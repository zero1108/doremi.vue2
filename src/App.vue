<template>
    <div id="app">
        <div id="background" :style="{backgroundImage:'url('+backgroundUrl+')'}"></div>
        <player :data="$data" ref="player" v-on:getLrc="getLrc"></player>
        <div ref="lrcboard" class="lrcboard" :class="{blur:isSearch&&!secondScreen,zoom:secondScreen&&alwaysLrc}" id="lrcboard">
            <ul ref="lrc" id="lrc">
                <li v-if="!lrc.result" style=" line-height: 1000%;">ヽ(*´∀｀*)ノ.+ﾟおはよ～♪.+ﾟ</li>
                <li :class="{active:$index==lrc.now-1}" v-for="(x,$index) in lrc.result">
                    <div class="lrc">{{x[1] +" "}}</div>
                    <div class="tLrc" v-if="lrc.tShow&&lrc.tResult">{{ lrc.tResult&&lrc.tResult[x[0]] }}</div>
                </li>
            </ul>
        </div>

        <div class="songInfo">
            <div class="songTitle">
                <h3 title="歌名">{{nowPlaying.name}}</h3>
            </div>
            <div class="songSinger">
                <ul style="list-style: none">
                    <li title="歌手" v-for="x in nowPlaying.ar||nowPlaying.artists">
                        {{x.name}}
                    </li>
                </ul>
            </div>
        </div>

        <div class="searchBoard" :class="{active:isSearch,zoom:secondScreen&&alwaysLrc}">
            <div class="closeSearch" v-if="isSearch" @click="closeSearch">
                <i class="fa fa-arrow-left"></i>
            </div>
            <div v-if="search.result" class="prevSearchPage" @click="prevSearchPage"><i class="fa  fa-chevron-left"></i></div>
            <div v-if="search.result" class="nextSearchPage" @click="nextSearchPage"><i class="fa  fa-chevron-right"></i></div>
            <div class="header" :class="{'active':isSearch}">
                <h2 class="domain"> doremi.moe </h2>
            </div>
            <div class="search" :class="{'active':isSearch}">
                <form id="search" @submit.prevent="doSearch(0)">
                    <input @focus="search.show=true" autofocus="autofocus" @blur="search.show=false" name="text" v-model="search.input" class="searchInput" id="searchInput" placeholder="这里搜索噢～">
                    <label for="searchInput"></label>

                    <label for="closeSearch">
                        <i class="fa fa-search"></i>
                        <input id="closeSearch" type="submit" style="display: none" />
                    </label>
                </form>
            </div>

            <div id="searchResult" class="body">
                <transition-group name="searchResultAnimation">
                    <div v-if="search.songs" v-for="(x,$index) in search.songs" :key="'num_'+$index" class="result searchResultAnimation" :class="{small:secondScreen&&alwaysLrc}" :style="{transition:'all '+($index+1)*0.2+'s'}">
                        <div class="coverImg">
                            <img width="100%" :src="x.album.picUrl">
                        </div>
                        <div class="resultTitle">
                            <h3>{{x.name}}</h3>
                            <span class="ctrl">
                              <span>
                                <i class="fa fa-heart-o"></i>
                              </span>
                              <span>
                                <i class="fa fa-star-o"></i>
                              </span>
                              <span>
                                <i @click="setPlay(x)" class="fa fa-play"></i>
                              </span>
                            </span>
                        </div>
                        <div class="resultDetail">
                            <span class="singer">
                                <span>歌手：</span><span style="padding-right:5px" v-for="s in x.artists">{{s.name}}</span>
                            </span>
                            <span class="album" :title="x.album.name">
                              专辑：album{{x.album.name}}
                            </span>
                        </div>

                    </div>

                    <div v-show="isSearch&&search.doing" :key="'searching'" class="result" style="text-align: center">搜索中...</div>

                </transition-group>
            </div>
        </div>


    </div>
</template>

<script>
    "use strict";
    import player from './components/playerController.vue';
    import commentMusic from './components/commentMusic.vue';
    export default {
        components:{
            player
        },
        name: 'app',
        data:function () {
            return {
                audio:document.createElement("audio"),
                backgroundUrl:"./src/img/default.jpg",
                randomList:[],
                search:{
                    show:false,
                    result:"",
                    songs:{},
                    input:"",
                    doing:false,
                    nowPage:0,
                    totalPage:0
                },
                alwaysLrc:true,
                secondScreen:false,
                isPlay:false,
                currentTime:{
                    original:0,
                    real:"00:00"
                },
                duration:{
                    original:0,
                    real:"00:00"
                },
                lrc:{
                    tShow:true,//是否显示翻译
                    tResult:"",//翻译的歌词
                    result:"",
                    now:0,
                    dom:''
                },
                autoplay:false,
                nowPlaying:'',
                mp3Url:"./src/demo.mp3",
                radio: '1',
                coverUrl:"./src/img/default.jpg",
                items: [1,2,3,4,5,6,7,8,9]
            }
        },
        mounted:function(){
            let that = this;
            setInterval(this.updateLrc,100);
            this.$http.get("/playlist/detail?id=324617415",{})
                .then(function(data){

                    this.randomList = data.data.playlist.tracks;

                },function(data){

                });
            this.audio.addEventListener("play",function(data){
            });
            this.audio.addEventListener("ended",that.playNext);
        },
        computed:{
            isSearch:function(){
                return this.secondScreen = this.search.show||this.search.input||this.search.result;
            }
        },
        watch:{
            audio:function(){
                this.currentTime.original=this.audio.currentTime;
                this.currentTime.real = parseInt(this.audio.currentTime/60)+this.audio.currentTime%60;
                this.duration.original=this.audio.duration;
                this.duration.real = ((this.audio.duration/60)<10?"0":"")+parseInt(this.audio.duration/60)+this.audio.duration%60;
            }
        },
        methods:{
            closeSearch:function(){
                this.search.show=false;
                this.search.input="";
                this.search.result='';
                this.search.songs='';
                this.secondScreen=false;
            },
            setPlay:function(ele,url){
                this.$refs.player.setPlay(ele,url);
            },
            play:function(url){
                this.$refs.player.play(url);
            },
            pause:function(){
                this.$refs.player.pause();
            },
            playNext:function(){
                this.$refs.player.playNext();
            },
            doSearch:function(page){
                this.search.nowPage = page||0;
                this.search.doing=true;
                this.search.songs={};
                let that = this;
                this.$http.get('/search?keywords='+this.search.input+"&offset="+this.search.nowPage*10+"&limit=10",{})
                    .then(function(res){
                        that.search.result=res.data;
                        that.search.songs=res.data.result.songs;
                        that.search.doing=false;
                    },function(res){
                        console.info("error");
                        that.search.doing=false;
                    })

            },
            prevSearchPage:function(){

                if(this.search.nowPage>0){
                    this.doSearch(--this.search.nowPage);
                }

            },
            nextSearchPage:function(){
                this.doSearch(++this.search.nowPage);


            },
            getLrc:function(id){
                id = id ||this.nowPlaying.id;
                this.$http.get('/lyric?id='+id,{})
                    .then(function(res){
                        if(res.data.code==200){
                            let lrc = res.data.lrc&&res.data.lrc.lyric;
                            let tLrc = res.data.tlyric&&res.data.tlyric.lyric;
                            this.lrc.result = this.parseLrc(lrc);
                            tLrc = this.parseLrc(tLrc) || "";
                            this.parseTLrc(tLrc&&tLrc);
                        }
                    },function(res){
                        console.log('error')
                    })
            },
            parseTLrc:function(lrc){
                let tLrc = {};
                lrc.forEach(function(el,id){
                    tLrc[el[0]] = el[1];
                });
                this.lrc.tResult = tLrc;
            },
            parseLrc:function(text){
                try{
                    let lines = text.split('\n'),
                        pattern = /\[\d{2}:\d{2}.\d{2}\]/g,
                        pattern2 = /\[\d{2}:\d{2}.\d{3}\]/g,
                        result = [];
                    result.splice(0,result.length);
                    let max_lrc_row = 0;


                    for(let i=0;i<=lines.length;i++){
                        if (pattern2.test(lines[i])){
                            lines[i] = lines[i].substring(0, 9) + lines[i].substring(10);
                            i--;
                        }
                    }
                    while (!pattern.test(lines[0])) {
                        lines = lines.slice(1);
                        max_lrc_row += 1;
                        if (max_lrc_row > 10000) {
                            console.log("歌词超过10000！！");
                            break;
                        }
                    }
                    lines[lines.length - 1].length === 0 && lines.pop();

                    for(let i=lines.length-1;i!=1;i--){

                        if(!lines[i][1]){

                            lines.pop();

                        }
                        else {break;}

                    }

                    lines.forEach(function(v  , i  , a  ) {
                        pattern.lastIndex=0;
                        pattern2.lastIndex=0;
                        if (pattern.test(lines[i])||pattern2.test(lines[i])){
                            let time = v.match(pattern),
                                value = v.replace(pattern, '');
                            time.forEach(function (v1, i1, a1) {
                                let t = v1.slice(1, -1).split(':');
                                result.push([parseInt(t[0], 10) * 60 + parseFloat(t[1]), value]);
                            });
                        }
                    });

                    result.sort(function(a, b) {
                        return a[0] - b[0];
                    });


                    let riki = [];
                    let desu = [];
                    let rikidesu = [];
                    riki.splice(0,riki.length);

                    result.forEach(function(v  , i  , a ){
                        let Static = /\[static\].*\[\/static\]/g,
                            effect = /\[effect\].*\[\/effect\]/g,
                            godlrc = /\[\{.*\}\]/g;
                        riki[i] = [];
                        desu[i] = [];
                        a[i][3] = [];
                        a[i][3][0] = 0;

                        if(godlrc.test(v)){

                            if(Static.test(v)){
                                a[i][3] = [];
                                a[i][2] = a[i][1].match(Static);
                                a[i][2] = String(a[i][2]).slice(+8,-9);
                                a[i][1] = a[i][1].replace(Static,"");

                                riki[i] = String(a[i][2]).split(";");

                                for(let j=0;j<riki[i].length;j++){

                                    a[i][3][j]= String(riki[i][j]).split(":");

                                }



                            }

                            a[i][1] = a[i][1].replace(godlrc,"");
                        }
                    });
                    //this.lrc.result = result;
                    result.unshift([0,'']);
                    result.push([99999,''],[99999,'']);
                    return result
                }catch(e){

                    console.info(e)

                }
            },
            updateLrc:function(){
                let that = this;
                this.isPlay = !this.audio.paused;
                this.currentTime.original=this.audio.currentTime;
                this.currentTime.real = ((this.audio.currentTime/60)<10?"0":"")+parseInt(this.audio.currentTime/60)+":"+(parseInt(this.audio.currentTime%60)<10?"0":"")+parseInt(this.audio.currentTime%60);
                this.duration.original=this.audio.duration;
                this.duration.real = ((this.audio.duration/60)<10?"0":"")+parseInt(this.audio.duration/60)+":"+(parseInt(this.audio.duration%60)<10?"0":"")+parseInt(this.audio.duration%60);
                if(this.lrc.result){

                    for(let i=this.lrc.now;i<this.lrc.result.length;i++){
                        if(this.audio.currentTime>=this.lrc.result[i][0]){
                            this.lrc.now=i+1;
                            try{
                            let el = this.$refs.lrcboard;
                            let from = this.$refs.lrcboard.scrollTop;
                            let toEl = this.$refs.lrc.querySelector(".active");
                            let to = toEl.offsetTop-this.$refs.lrcboard.clientHeight/2+toEl.clientHeight*3;
                            let dur = (to-from)/30;
                            let scrollTo = function(){
                                from+=dur;
                                el.scrollTop = from;
                                if(dur>0&&from<to){
                                    requestAnimationFrame(scrollTo);
                                }else if(dur<0&&from>to){
                                    requestAnimationFrame(scrollTo);
                                }
                            };

                            requestAnimationFrame(scrollTo);

                            //this.$refs.lrcboard.scrollTop = this.$refs.lrc.querySelector(".active").offsetTop;


                            }catch(e){

                            }

                            break;
                        } else if(this.audio.currentTime<=0.5){
                            that.lrc.now=0;
                            that.$refs.lrcboard.scrollTop=0;
                        }
                    }
                }
            }
        }
    }
</script>

<style>
    #app{
        height:100%;
        overflow-y: auto;
        color:#404d5b;
        font-family: "Segoe UI", "Lucida Grande", Helvetica, Arial, "Microsoft YaHei", FreeSans, Arimo, "Droid Sans", "wenquanyi micro hei", "Hiragino Sans GB", "Hiragino Sans GB W3", "FontAwesome", sans-serif;
    }
    #background{
        position:fixed;
        width:100%;
        height:100%;
        background-size: cover;
        z-index: -1;
        -webkit-filter: blur(15px); /* Chrome, Opera */
        -moz-filter: blur(15px);
        -ms-filter: blur(15px);
        filter: blur(15px);
        opacity:.4;
    }
    .searchResultAnimation{
        opacity:1;
    }
    .searchResultAnimation-enter{
        transform: scale(.8) rotateY(90deg);
        opacity:0;
    }
    .searchResultAnimation-enter-active{
    }
    .searchResultAnimation-leave{
        opacity:0;
    }

    .searchResultAnimation-leave-active{
        transition: all .3s!important;
        opacity:0;
    }
    .prevSearchPage{
        position:absolute;
        left:0;
        top:0;
        height:100%;
        font-size:100px;
        width:100px;
        cursor:pointer;
    }
    .nextSearchPage{
        position:absolute;
        right:0;
        top:0;
        height:100%;
        font-size:100px;
        width:100px;
        cursor:pointer;
    }
    .prevSearchPage i,.nextSearchPage i{
        position:absolute;
        top:50%;
        margin-top:-80px;
    }
    html,body{
        margin:0;
        padding:0;
        height:100%;
    }
    body{
        background-color: #F5F5F5;
    }
    *::-webkit-scrollbar-thumb{
        background-color:rgba(102,102,102,.6);
        color:red;
        height:6px;
        width:6px;
        border-radius:10px;
    }
    *::-webkit-scrollbar{
        width:8px;
        height:8px;

    }
    /*.side{*/
        /*position:fixed;*/
        /*bottom:20px;*/
        /*left:30px;*/
        /*z-index: 10;*/
    /*}*/
    /*.cover{*/
        /*position:relative;*/
        /*margin:0 auto;*/
        /*width:160px;*/
        /*height:160px;*/
        /*border-radius:100%;*/
        /*border:solid 4px #fff;*/
        /*overflow:hidden;*/
        /*box-shadow:0 0 3px 3px rgba(51,51,51,.06);*/
    /*}*/
    /*.progress{*/
        /*position:absolute;*/
        /*width:100%;*/
        /*height:100%;*/
        /*top:0;*/
        /*left:0;*/
        /*border-radius:100%;*/
    /*}*/
    /*.progress i{*/
        /*position: absolute;*/
        /*color: #111;*/
        /*font-size: 20px;*/
        /*margin-left:-10px;*/
        /*top: -20px;*/
        /*left: 50%;*/
    /*}*/
    .lrcboard{
        position: absolute;
        top:0;
        left:0;
        width:100%;
        height:100%;
        overflow-y: auto;
        overflow-x: hidden;
        text-align:center;
        font-size: 28px;
        background-color:rgba(255,255,255,.3);
        text-shadow: 0 0 6px rgba(0,0,0,.4);
        -webkit-transition-duration: .6s;
        -moz-transition-duration: .6s;
        -ms-transition-duration: .6s;
        -o-transition-duration: .6s;
        transition-duration: .6s;
        transition-delay: .3s;
    }
    .lrcboard.zoom{
        width:40%;
        font-size:18px;
        border-right:solid 1px #666
    }
    .lrcboard.blur{
        -webkit-filter: blur(10px); /* Chrome, Opera */
        -moz-filter: blur(10px);
        -ms-filter: blur(10px);
        filter: blur(10px);
    }
    ul#lrc {
        padding-left:50px;
        padding-right:50px;
        -webkit-transform-style: preserve-3d;
        -moz-transform-style: preserve-3d;
        -ms-transform-style: preserve-3d;
        transform-style: preserve-3d;
        backface-visibility: hidden;
        list-style: none;
    }
    .lrcboard li{
        margin:32px 0 32px 0;
        /*white-space:pre-wrap;*/
        -webkit-transition-duration: .6s;
        -moz-transition-duration: .6s;
        -ms-transition-duration: .6s;
        -o-transition-duration: .6s;
        transition-duration: .6s;
    }
    .lrc {
        white-space: pre;
    }
    .tLrc {
        position:absolute;
        left:0;
        right:0;
        font-size:14px;
        color:rgba(64, 77, 91, 0.8)
    }
    .lrcboard li.active .tLrc{
        color:#6698ff;
    }
    /*
    .lrcboard li.hidden{
        opacity: 0;
    }
    */
    .lrcboard li.active{
        willchange:transform;
        -webkit-transform: scale(1.1);
        -moz-transform: scale(1.1);
        -ms-transform: scale(1.1);
        -o-transform: scale(1.1);
        transform: scale(1.1);
        color:#6698ff;
        text-shadow: 0 0 6px #6698ff;
    }
    .songInfo{
        position: absolute;
        right: 30px;
        text-align: right;
    }
    .songTitle{

    }
    .songSinger{
        text-align: right;

    }
    .searchBoard{
        position:absolute;
        height:100%;
        width:100%;
        overflow: hidden;
        overflow-y: auto;
        transition:all .5s;
        pointer-events: none;
    }
    .searchBoard.active{
        background-color: rgba(153,153,153,.2);
        pointer-events: auto;
    }
    .searchBoard.zoom{
        width:60%;
        right:0;
    }
    .header{
        position:relative;;;
        text-align: center;
        padding:5px 20px;
        transition: all .6s;
        opacity: 0;
    }
    .header.active{
        will-change:transform;
        transform: translateY(0) scale(1);
        opacity:1;
    }
    .domain{
        font-size: x-large;
    }
    .search{
        position:relative;
        width:200px;
        padding:5px 10px;
        transition: all .6s;
        left:0;
        transform: translateY(-70px) scale(1);
        pointer-events: auto;
    }
    .closeSearch{
        position: absolute;
        top: 0;
        left: 0;
        z-index: 2;
        width: 100px;
        height: 80px;
        line-height: 80px;
        text-align: center;
        background-color: rgba(0, 0, 0, .6);
        font-size: 60px;
        color: #fff;
        cursor: pointer
    }
    .search.active{
        will-change:transform,left;
        left:calc(50% - 210px);
        width:400px;
        transform: translateY(0) scale(1.2);
    }
    .search i.fa-search{
        position:absolute;
        right:-2px;
        bottom:15px;
        font-size: 24px;
        color:#666666;
        cursor: pointer;
    }
    .searchInput{
        width:100%;
        height:30px;
        padding:5px 10px;
        font-weight:normal;
        font-size:18px;
        border:none;
        border-bottom:solid 2px #66ccff;
        outline: none;
        background-color: transparent;

    }
    .body{
        width:900px;
        min-height:400px;
        height:auto;
        margin:30px auto;
        border-radius:4px;
        border:solid 1px transparent;
        right:0;
        /*box-shadow: 1px 1px 1px 1px rgba(153,153,153,.2);
        background-color: #fff;*/
    }

    .results{
        position:relative;
        margin:10px auto;
        width:800px;
        height:120px;
        background-color: #fff;
        border-radius:3px;
        border:solid 1px rgba(204,204,204,.2);
        box-shadow:2px 2px 1px 1px rgba(204,204,204,.3);
    }
    .result{
        position:relative;
        margin:30px auto;
        width:800px;
        height:110px;
    }
    .result.small{
        width:600px;
    }
    .result .coverImg{
        position:absolute;
        bottom:-2px;
        left:-60px;
        width:120px;
        height:120px;
        overflow: hidden;
        border-radius: 50%;
        border:solid 2px rgba(222,222,222,.8);
        background-color: rgba(244,244,244,1);
    }
    .resultTitle{
        position:relative;
        margin-left:80px;
    }
    .resultTitle .ctrl {
        position:absolute;
        top:5px;
        right:5px;
    }
    .resultTitle .ctrl span{
        display: inline-block;
        margin:0 15px;
        font-size: larger;
        color:#41a1ff;
        cursor:pointer;
    }
    .resultDetail{
        height:35px;
        line-height: 35px;
        padding-left:100px;
        border-radius: 5px;
        background-color: rgba(251,251,251,.8);
        border:solid 1px rgba(222,222,222,.8);
        box-shadow: 0 0 1px 1px rgba(222,222,222,.8);
    }
    .resultDetail .singer{
        position:relative;
        width:50%;
        white-space:nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        display: inline-block;
    }
    .resultDetail .singer::before{
        position:absolute;
        top:15%;
        left: -15px;
        height:70%;
        width:4px;
        background-color: #66ccff;
        content: "";
    }
    .resultDetail .album{
        position:relative;
        width:45%;
        white-space:nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        display: inline-block;
    }
    .resultDetail .album::before{
        position:absolute;
        top:15%;
        left: -15px;
        height:70%;
        width:4px;
        background-color: #66ffcc;
        content: "";
    }
    .rCover{
        margin-left:1px;
        width:100px;
        height:100px;
        display: inline-block;
        box-shadow:4px 0 0 0 rgba(102,204,255,.5);
    }
    .rCover img{
        width:100px;
        height:100px;
    }
    .rDetail{
        position:relative;
        overflow: hidden;
        width:650px;
        height:100%;
        display: inline-block;
        vertical-align: top;
        padding-left:20px;
    }
    .rDetail .title{
        font-size: large;
    }
    .rCtrl{
        position:absolute;
        top:20px;
        right:40px;
        font-size:80px;
        color:#666;
    }
    /*.controller{*/
        /*position:absolute;*/
        /*top:0;*/
        /*width:100%;*/
        /*height:100%;*/
        /*bottom:10px;*/
        /*opacity: 0;*/
        /*border-radius:100%;*/
        /*overflow: hidden;*/
        /*transform: scale(1.2) translateY(-5px);*/
        /*transition: all .3s;*/
    /*}*/
    /*.play{*/
        /*position:relative;*/
        /*margin:25px auto -15px auto;*/
        /*text-align: center;*/
        /*color:#FFF;*/
        /*font-size: 50px;*/
        /*cursor:pointer;*/
        /*text-shadow: 0 0 3px rgba(51,51,51,.6);*/
    /*}*/
    /*.coverModel{*/
        /*position: absolute;*/
        /*top: 80px;*/
        /*width: 380px;*/
        /*left: -111px;*/
        /*height: 300px;*/
        /*border-radius: 100%;*/
        /*background-color: rgba(102,204,255,0.7);*/
    /*}*/
    /*.playingImg{*/
        /*height: 100%;*/
        /*width: 100%;*/
        /*background-repeat: no-repeat;*/
        /*background-size: cover;*/
    /*}*/
    /*.time{*/
        /*position:relative;*/
        /*top:14px;*/
        /*height:22px;*/
        /*color:#eee;*/
        /*font-size:small;*/
        /*text-align: center;*/
    /*}*/
    /*.time div:first-child{*/
        /*margin-bottom:2px;*/
        /*display: inline-block;*/
        /*border-bottom: solid 2px rgba(255,204,255,0.8);*/
    /*}*/
    /*.other{*/
        /*position:relative;*/
        /*margin:0 auto 10px auto;*/
        /*text-align: center;*/
        /*color:#FFF;*/
        /*font-size: 26px;*/
        /*text-shadow: 0 0 3px rgba(51,51,51,.6);*/
    /*}*/
    /*.other i{*/
        /*margin:10px;*/
        /*position:relative;*/
        /*cursor: pointer;*/
    /*}*/
    /*.other .fa-random{*/
        /*top:30px;*/
    /*}*/
    /*.other .fa-heart{*/
    /*}*/
    /*.other .fa-forward{*/
    /*}*/


    .suki{
        position: relative;
        text-align: center;
        cursor: pointer;
        top:-85px;
        margin-right:1px;
        width: 50px;
        height: 50px;
        display: inline-block;
    }
    .next{
        position: relative;
        text-align: center;
        cursor: pointer;
        top:-25px;
        width: 50px;
        height: 50px;
        display: inline-block;
    }
    .kirai {
        position: relative;
        width: 50px;
        height: 50px;
        top:-85px;
        margin-left:1px;
        cursor: pointer;
        display: inline-block;
    }
    .suki img{
        -webkit-transition: transform .5s,opacity .5s;
        -moz-transition: transform .5s,opacity .5s;
        -ms-transition: transform .5s,opacity .5s;
        -o-transition: transform .5s,opacity .5s;
        transition: transform .5s,opacity .5s;
    }
    .suki img:nth-child(1){
        position: absolute;
        top:0;
        left:0;
        z-index: 2;
    }
    .suki.hover img:nth-child(1){
        opacity: 0;
    }
    .suki.sukidesu img:nth-child(1) {
        opacity: 0;
    }
    .suki.sukidesu img:nth-child(2) {
        opacity: 1;
    }
    .suki img:nth-child(2){
        position: absolute;
        top:0;
        left:0;
        z-index: 1;
        opacity: 0;
    }
    .suki img:nth-child(5){
        position: absolute;
        top:0;
        left:0;
        z-index: 3;
        opacity: 0;
    }
    .suki.click img:nth-child(5){
        -webkit-animation: sukiclick 3s linear;
        -o-animation: sukiclick 3s linear;
        animation: sukiclick 3s linear;
    }
    .suki.hover img:nth-child(2){
        -webkit-transform: scale(1.1);
        -moz-transform: scale(1.1);
        -ms-transform: scale(1.1);
        -o-transform: scale(1.1);
        transform: scale(1.1);
        opacity: 1;
    }
    .kirai .mid{
        position: absolute;
        width: 50px;
        height: 50px;
    }
    .kirai .right,.left{
        -webkit-transition: transform 1s,opacity .5s;
        -moz-transition: transform 1s,opacity .5s;
        -ms-transition: transform 1s,opacity .5s;
        -o-transition: transform 1s,opacity .5s;
        transition: transform 1s,opacity .5s;
        position: absolute;
        top: 0;
        left: 0;
        width: 50px;
        height: 50px;
    }
    .kirai.hover .left{
        -webkit-transform: rotateZ(-15deg) translateX(-5px);
        -moz-transform: rotateZ(-15deg) translateX(-5px);
        -ms-transform: rotateZ(-15deg) translateX(-5px);
        -o-transform: rotateZ(-15deg) translateX(-5px);
        transform: rotateZ(-15deg) translateX(-5px);
    }
    .kirai.hover .right{
        -webkit-transform: rotateZ(15deg) translateX(5px);
        -moz-transform: rotateZ(15deg) translateX(5px);
        -ms-transform: rotateZ(15deg) translateX(5px);
        -o-transform: rotateZ(15deg) translateX(5px);
        transform: rotateZ(15deg) translateX(5px);
    }
    .kirai.click .left{
        -webkit-animation: kiraiclickL 3s ;
        -o-animation: kiraiclickL 3s ;
        animation: kiraiclickL 3s ;
    }
    .kirai.click .right{
        -webkit-animation: kiraiclickR 3s ;
        -o-animation: kiraiclickR 3s ;
        animation: kiraiclickR 3s ;
    }
    .kirai.click .mid  {
        -webkit-animation: kiraiclickM 3s linear;
        -o-animation: kiraiclickM 3s  linear;
        animation: kiraiclickM 3s  linear;
    }
    .kirai.hover .left span,.kirai.hover .right span{
        opacity: .8;
    }
    .kirai.hover .mid span{
        opacity: .8;
        top:-5px;
    }
    .suki span:nth-child(3){
        -webkit-transition:opacity .8s;
        -moz-transition:opacity .8s;
        -ms-transition: opacity .8s;
        -o-transition: opacity .8s;
        transition:opacity .8s;
        position: absolute;
        line-height: 40px;
        color:rgba(0,0,0,.6);
        top:0;
        left:5px;
        opacity: 0;
    }
    .suki.hover span:nth-child(3){
        opacity: 1;
        z-index: 3;
    }
    .suki span:nth-child(4){
        -webkit-transition:opacity .8s;
        -moz-transition:opacity .8s;
        -ms-transition: opacity .8s;
        -o-transition: opacity .8s;
        transition:opacity .8s;
        position: absolute;
        line-height: 40px;
        color:rgba(0,0,0,.6);
        top:0;
        right:5px;
        opacity: 0;
    }
    .suki.hover span:nth-child(4){
        opacity: 1;
        z-index: 3;
    }
    .kirai span span{
        -webkit-transition: opacity .5s,top .5s;
        -moz-transition: opacity .5s,top .5s;
        -ms-transition: opacity .5s,top .5s;
        -o-transition: opacity .5s,top .5s;
        transition: opacity .5s,top .5s;
        color:rgba(0,0,0,.6);
        position: absolute;
        line-height: 40px;
        opacity: 0;
    }
    .kirai .left span{
        top:0;
        left: 5px;
    }
    .kirai .mid span{
        top:0;
        left: 16px;
    }
    .kirai .right span{
        top:0;
        right: 5px;
    }


    @keyframes kiraiclickL
    {
        30%{
            -webkit-transform: rotateZ(-40deg) translateX(-25px) translateY(10px);
            -moz-transform: rotateZ(-40deg) translateX(-25px) translateY(10px);
            -ms-transform: rotateZ(-40deg) translateX(-25px) translateY(10px);
            -o-transform: rotateZ(-40deg) translateX(-25px) translateY(10px);
            transform: rotateZ(-40deg) translateX(-25px) translateY(10px);
            opacity: 0;
        }
        50%{
            opacity: 0;
        }
        80%{
            -webkit-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            -moz-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            -ms-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            -o-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            transform: rotateZ(0deg) translateX(0px) translateY(0px);
        }

        100%{
            opacity: 1;
        }
    }
    @keyframes kiraiclickM
    {
        20%{
            opacity: 0;
        }
        85%{
            -webkit-transform: rotateZ(1440deg) translateY(80px) scale(0);
            -moz-transform: rotateZ(720deg) translateY(80px) scale(0);
            -ms-transform: rotateZ(720deg) translateY(80px) scale(0);
            -o-transform: rotateZ(720deg) translateY(80px) scale(0);
            transform: rotateZ(1440deg) translateY(80px) scale(0);
        }
        99%{
            -webkit-transform: rotateZ(0deg) translateY(0px) scale(1);
            -moz-transform: rotateZ(0deg) translateY(0px) scale(1);
            -ms-transform: rotateZ(0deg) translateY(0px) scale(1);
            -o-transform: rotateZ(0deg) translateY(0px) scale(1);
            transform: rotateZ(0deg) translateY(0px) scale(1);
            opacity: 0;
        }

        100%{
            opacity: 1;
        }
    }
    @keyframes kiraiclickR
    {
        30%{
            -webkit-transform: rotateZ(-40deg) translateX(25px) translateY(10px);
            -moz-transform: rotateZ(40deg) translateX(25px) translateY(10px);
            -ms-transform: rotateZ(40deg) translateX(25px) translateY(10px);
            -o-transform: rotateZ(40deg) translateX(25px) translateY(10px);
            transform: rotateZ(40deg) translateX(25px) translateY(10px);
            opacity: 0;
        }
        50%{
            opacity: 0;
        }
        80%{
            -webkit-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            -moz-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            -ms-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            -o-transform: rotateZ(0deg) translateX(0px) translateY(0px);
            transform: rotateZ(0deg) translateX(0px) translateY(0px);
        }

        100%{
            opacity: 1;
        }
    }
    @keyframes sukiclick
    {
        20%{
            -webkit-transform:  scale(1.4);
            -moz-transform:  scale(1.4);
            -ms-transform:  scale(1.4);
            -o-transform:  scale(1.4);
            transform: scale(1.4);
            opacity: 1;
        }
        35%{
            -webkit-transform:rotateY(0deg) translateY(0px) scale(1.4);
            -moz-transform:rotateY(0deg) translateY(0px) scale(1.4);
            -ms-transform:rotateY(0deg) translateY(0px) scale(1.4);
            -o-transform:rotateY(0deg) translateY(0px) scale(1.4);
            transform:rotateY(0deg) translateY(0px) scale(1.4);
            opacity: 1;
        }
        50%{
            -webkit-transform:rotateY(180deg) translateY(-30px) scale(.3);
            -moz-transform:rotateY(180deg) translateY(-30px) scale(.3);
            -ms-transform:rotateY(180deg) translateY(-30px) scale(.3);
            -o-transform:rotateY(180deg) translateY(-30px) scale(.3);
            transform:rotateY(180deg) translateY(-30px) scale(.3);
        }
        60%{
            -webkit-transform:rotateY(360deg) translateY(-25px)  scale(.1);
            -moz-transform:rotateY(360deg) translateY(-25px)  scale(.1);
            -ms-transform:rotateY(360deg) translateY(-25px)  scale(.1);
            -o-transform:rotateY(360deg) translateY(-25px)  scale(.1);
            transform:rotateY(360deg) translateY(-25px)  scale(.1);
        }
        80%{
            -webkit-transform: rotateY(540deg) translateY(-50px)  scale(.1);
            -moz-transform: rotateY(540deg) translateY(-50px) scale(.1);
            -ms-transform: rotateY(540deg) translateY(-50px) scale(.1);
            -o-transform: rotateY(540deg) translateY(-50px)  scale(.1);
            transform: rotateY(540deg) translateY(-50px)  scale(.1);
            opacity: 0;
        }
        99%{
            -webkit-transform: rotateY(540deg) translateY(-70px)  scale(.1);
            -moz-transform: rotateY(540deg) translateY(-70px) scale(.1);
            -ms-transform: rotateY(540deg) translateY(-70px) scale(.1);
            -o-transform: rotateY(540deg) translateY(-70px)  scale(.1);
            transform: rotateY(540deg) translateY(-70px)  scale(.1);
            opacity: 0;
        }
        100%{
            -webkit-transform: rotateZ(0deg) translateX(0px) translateY(0px) scale(1);
            -moz-transform: rotateZ(0deg) translateX(0px) translateY(0px) scale(1);
            -ms-transform: rotateZ(0deg) translateX(0px) translateY(0px) scale(1);
            -o-transform: rotateZ(0deg) translateX(0px) translateY(0px) scale(1);
            transform: rotateZ(0deg) translateX(0px) translateY(0px) scale(1);
        }

    }

</style>