<template>

  <div class="SongListDetail scrollStyle" ref="SongListDetail">
      <header class="SongListDetailHeader">
          <div class="listPic bgc" :style="{backgroundImage:`url(${detail.coverImgUrl})`}"></div>
          <div class="listDetailContainer">
              <div class="titleContainer alignCenter">
                  <span class="SongListsign">歌单</span>
                  <p class="mainTitle">{{detail.name}}</p>
                  <div class="numData">
                      <p class="title">歌曲数</p>
                      <p class="num">{{detail.trackCount}}</p>
                  </div>
                  <div class="numData">
                      <p class="title">播放数</p>
                      <p class="num">{{detail.playCount | wan}}</p>
                  </div>
              </div>
              <div class="avatar alignCenter">
                  <div class="avaPic bgc" :style="{'backgroundImage':`url(${detail.creator.avatarUrl})`}"></div>
                  <p class="avaName">{{detail.creator.nickname}}</p>
                  <p class="createTime">{{detail.createTime | middleTime}}创建</p>
              </div>
              <div class="controlContainer alignCenter">
                  <div class="item alignCenter">
                      <i class="iconfont icon-bofang" @click="playAll"></i><p @click="playAll">播放全部</p><i class="add">+</i>
                  </div>
                  <div class="item alignCenter" :plain="true" @click="collect" v-if="!isCollect">
                      <i class="iconfont icon-shoucanggedan"></i><p>收藏({{collectNum}})</p>
                  </div>
                  <div class="item alignCenter" v-if="isCollect" @click="canCollect" :plain="true">
                      <i class="iconfont icon-xiangxiayuanjiantouxiajiantouxiangxiamianxing"></i><p>已收藏({{collectNum}})</p>
                  </div>
                  <div class="item alignCenter">
                       <i class="iconfont icon-fenxiang"></i><p>分享({{detail.shareCount}})</p>
                  </div>
                  <div class="item alignCenter">
                        <i class="iconfont icon-xiazai"></i><p>下载全部</p>
                  </div>
              </div>
              <div class="tagcontainer alignCenter">
                  <p class="title">标签</p>
                  <div class="tagcontents">
                    <span class="tagcontent" v-for="item in detail.tags" :key="item">&nbsp;<b class="tag">{{item}}</b>&nbsp;/</span>
                  </div>  
              </div>
              <div class="abstruct spCenter" :class="dropCls">
                    <ul class="abstructContent">
                      简介：<li class="abItem" v-for="(item,index) in description" :key="index" v-html="item">
                      </li>
                    </ul>
                  <i class="iconfont" :class="showCls" @click="wordHide = !wordHide"></i>
              </div>
          </div>
      </header>
      <main class="songListMain spCenter">
          <nav class="songListNav alignCenter">
              <div v-for="(item,index) in navList" :key = "item" @click="cur = index" class="Navitem" :class="{'NavActive':cur == index}">{{item}}</div>
          </nav>
          <div class="searchSong" v-if="cur == 0">
              <input type="text" class="searchBox" placeholder="搜索歌单音乐" v-model="searchContent" @keyup="search">
              <i class="iconfont icon-sousuo" @click="search"></i>
          </div>
      </main>
      <div class="table alignCenter" v-if="cur == 0">
          <div class="tableItem tControl">操作</div>
          <div class="tableItem tMusicTitle">音乐标题</div>
          <div class="tableItem tSinger">歌手</div>
          <div class="tableItem tAlbum">专辑</div>
          <div class="tableItem tDuration">时长</div>
      </div>
      
      <album :Songs="songListc" :show="cur == 0" :types="1" :width='100' :nameWidth="32" v-if="songList.length > 0" :key="songListc.length">

      </album>

      <div class="sldCommentContainer" v-if="cur == 1">
          <input class="sldComment" type="text" v-model="commentContent" placeholder="140" @keyup.enter="sendComment"/>
          <div class="slcConmentIcon">
              <i class="iconfont icon-xiaolian"></i>
              <i class="iconfont">@</i>
              <i class="iconfont">#</i>
          </div>
          <button class="sendComment" @click="sendComment" :plain="true">评论</button>
      </div>
      
      <div class="comContainer" v-if="cur == 1">
          <p class="comtitle" ref = 'comment'>
              最新评论
          </p>
          <comment :list="commentList" :songListId="parseInt(this.$route.params.id)" @deleteComment="deleteCom" :key="commentList[0].user.userId"></comment>
          <el-pagination
                style="margin:20px 0 50px 50%;transform:translateX(-50%)"
                background
                :page-size = "20"
                layout="prev, pager, next"
                @current-change = "changePage"
                :total="total">
            </el-pagination>
      </div>
      <div class="collecter"  v-if="cur == 2">
          <ImaList :list="collecterList" :listWidth="20">
              <template v-slot:img="imgs">
                  <img v-lazy="imgs.imgs.avatarUrl" style="border-radius:50%;margin:0 auto;width:40%;display:block;">
              </template>
          </ImaList>
          <div v-loading="loading" v-if="loading"></div>
          <p style="width:100%;text-align:center;margin-bottom:50px;color:#888888;" v-if="!loading">没有更多收藏者了~~~</p>
      </div>
  </div>
</template>

<script>
import {createSong} from '../common/song'
import {initPage} from '../common/utils'
import {mapGetters,mapActions} from 'vuex'
import Album from '../base/Album'
import Comment from '../base/comment'
import ImaList from '../base/ImgList'
import {Axios,getSongListDetail,getComment,getSLCollecter,getSongUrl,collectSongList,sendComment} from '../common/api'
import {collectPlayList,setLoading} from '../common/mixin'

export default {
    mixins:[collectPlayList,setLoading],
    data() {
        return {
            description : [],
            songList : [],
            songListc : [],
            commentList:[],
            collecterList: [],
            detail: {},
            collectNum:0,
            limit:20,
            total:0,
            offset:0,
            loading: true,
            moreColl:true,
            searchContent:'',
            commentContent:'',
            wordHide: true,
            cur:0,
            navList: [
                '歌曲列表',
                '评论',
                '收藏者'
            ]
       }
    },
    components: {
        Album,
        Comment,
        ImaList
    },
    created() {
        this.initAll()
    },
    mounted() {
        this.init()
    },
    computed: {
        showCls() {
            return this.wordHide?'icon-down':'icon-shang'
        },
        dropCls() {
            return this.wordHide?'up':'drop'
        }
    },
    watch:{
        '$route'(to,from) {
            this.initAll()
        }
    },
    methods: {
        
        init() {
            setTimeout(() => {
                let outside = document.documentElement,
                innerside = this.$refs.SongListDetail
                initPage(outside,innerside)
            }, 500); 
        },
        initAll() {
            this.id = parseInt(this.$route.params.id)
            this.params = {
                id: this.id
            }
            this.initSongListDetail()
            this.initCommentList()
            this.initCollecter()
            this.moreCollecter()
        },
        initSongListDetail() {
            Axios(getSongListDetail,this.params).then((res) => {
                this.detail = res.playlist
                this.collectNum = res.playlist.subscribedCount
                let des = res.playlist.description
                if(des) {
                    des = des.split('\n')
                    des.forEach((item,index) => {
                        if(item == '') {
                            des[index] = '&#8197;'
                        }
                    })
                    this.description = des
                }
                this.songList = this._normalizeSongList(this.detail.tracks)
                this.songListc = this._normalizeSongList(this.detail.tracks) 
                this.set_loading(false)     
            })
        },
        initCommentList() {
            Axios(getComment,this.params).then((res) => {
                this.commentList = res.comments
                this.total = res.total
            })
        },

        initCollecter() {
            let params = {
                id: this.id,
                limit: 50
            }
            Axios(getSLCollecter,params).then((res) => {
                this.collecterList = res.subscribers
                this.moreColl = res.more
            })
        },
        moreCollecter() {
            setTimeout(() => {
                this.$refs.SongListDetail.onscroll = () => {
                    let clientHeight = this.$refs.SongListDetail.clientHeight,
                    scrollTop = this.$refs.SongListDetail.scrollTop,
                    scrollHeight = this.$refs.SongListDetail.scrollHeight
                    if(clientHeight + scrollTop >= scrollHeight && this.cur == 2 && this.moreColl) {
                        this.offset += 50
                        let params = {
                            id: this.id,
                            limit: 50,
                            offset: this.offset
                        }
                        Axios(getSLCollecter,params).then((res) => {
                            this.moreColl = res.more
                            
                            if(!this.moreColl) {
                                return this.loading = false
                            }
                            this.collecterList = this.collecterList.concat(res.subscribers)
                        })
                    }
                } 
            }, 500);           
        },
        _normalizeSongList(list) {
            let ret = []
            for(let i = 0; i < list.length; i ++) {
                ret.push(createSong(list[i]))
            }
            return ret
        },
        playAll() {
            /*标记*/
            let url = ''
            let index = Math.ceil(Math.random() * (this.songList.length-1))
            let params = {
                id:this.id
            }
            Axios(getSongUrl,params).then((res) => {
                url = res.data[0].url
                this.songListc[index].url = url
                this.selectPlay({
                   list:this.songListc,
                   index:index
                })
            })
        },
        search() {
            let ret = []
            for(let i = 0;i < this.songList.length; i ++) {
                if(this.songList[i].name.includes(this.searchContent)) {
                    ret.push(this.songList[i])
                }
            }

            this.songListc = ret
        },

        changePage(index) {
            let offset = (index-1) * this.limit
            let params = {
                id: this.id,
                offset: offset
            }
            Axios(getComment,params).then((res) => {
                    this.commentList = res.comments
                this.$refs.SongListDetail.scrollTop = this.$refs.comment.offsetTop - 10
            })
        },
        sendComment() {
            let params = {
                t: 1,
                type: 2,
                id: this.id,
                content: this.commentContent,
                timestamp: (new Date()).getTime()
            }
            Axios(sendComment,params).then((res) => {
                this.$message({
                    type:'success',
                    message: '发送成功',
                    center: true
                });
                this.commentContent = ''
                this.commentList.unshift(res.comment)
            })
        },
        deleteCom(id) {
            let index = this.commentList.findIndex((item) => {
                return item.commentId == id
            })
            this.commentList.splice(index,1)
        },
        ...mapActions([
            'selectPlay'
        ])
    }
}
</script>

<style lang='scss'>
@import '../assets/css/base.scss';
.SongListDetail {
    position: fixed;
    z-index: 99;
    left: 200px;
    top: 50px;
    width: 880px;
    height:590px;
    background: #FAFAFA;
    overflow: hidden;
    overflow-y: scroll;
    overflow-x: hidden;
	border-left: 1px solid $borderColor;
    .SongListDetailHeader {
        display: flex;
        margin: 30px;
        .listPic {
            width:200px;
            height: 200px;
        }
        .listDetailContainer {
            margin-left: 30px;
            box-sizing: border-box;
            .titleContainer {
                .SongListsign {
                    border:1px solid #E03F40;
                    color:#E03F40;
                    font-size: 13px;
                    padding: 3px 5px;
                }
                .mainTitle {
                    width: 400px;
                    font-size: 25px;
                    margin-left: 5px;
                    font-weight: 600;
                    color:black;
                }
                .numData {
                    font-size: 12px;
                    color: #555555;
                    padding: 0 10px;
                    border-left: 1px solid $borderColor;
                    position: absolute;
                    right: 10px;
                    &:nth-of-type(1){
                        border-left:none;
                        right: 70px;
                    }
                    .num {
                        float: right;
                        margin-top: 5px;
                    }
                }
            }
            .avatar {
                margin-top: 15px;
                .avaPic {
                    width: 30px;
                    height: 30px;
                    border-radius: 50%;
                    border: 1px solid #ccc;
                }
                .avaName {
                    margin-left: 10px;
                    font-size: 15px;
                    color:#444444;
                }
                .createTime {
                    font-size: 12px;
                    margin-left: 20px;
                    color:#444444;
                }
            }
            .controlContainer {
                margin-top: 20px;
                .item {
                    font-size: 13px;
                    margin-right: 15px;
                    border: 1px solid $borderColor;
                    padding: 5px 10px;
                    border-radius: 4px;
                    color:#555555;
                    &:hover {
                        color:#333333;
                    }
                    cursor: pointer;
                    i {
                        margin-right: 5px;
                    }
                    &:nth-of-type(1) {
                        background: rgb(222,35,35);
                        color: white;
                        padding-right: 5px;
                        .add {
                            font-size: 20px;
                            margin-left: 20px;
                        }
                        &:hover {
                            background: #B12323;
                        }
                    }
                }
            }
            .tagcontainer {
                font-size: 12px;
                margin-top: 25px;
                .title {
                    color:#555555;
                }
                .tagcontents {
                    margin-left: 10px;
                    .tagcontent {
                        .tag {
                            color:#0A63A8;
                        } 
                    }
                }
            }
            
            .abstruct {
                width: 500px;
                height: 32px;
                margin-top: 10px;
                font-size: 12px;
                color:#333333;
                overflow: hidden;
                .abstructContent {
                    width: 460px;
                    .icon-down {
                        margin-left: 100px;
                    }
                    .abItem{
                        line-height: 17px;
                        &:first-child{
                            display: inline-block;
                        }
                    }
                }
                
            }
            .drop {
                animation: drop .5s linear forwards;
            }
            .up {
                animation: up .5s linear reverse forwards;
            }
            @keyframes drop {
                0% {
                    height: 32px;
                }
                25% {
                    height: 25%;
                }
                50% {
                    height: 50%;
                }
                75% {
                    height: 75%;
                }
                100% {
                    height: 100%;
                }
            }
            @keyframes up {
                0% {
                    height: 32px;
                }
                25% {
                    height: 25%;
                }
                50% {
                    height: 50%;
                }
                75% {
                    height: 75%;
                }
                100% {
                    height: 100%;
                }
            }
        }
    }
    .songListMain {
        .songListNav {
            margin-left: 30px;
            border-bottom: 1px solid $borderColor;
            .Navitem {
                position: relative;
                padding: 10px 20px;
                font-size: 14px;
                color:#444444;
                cursor: pointer;
                &:hover{
                    color: #111111;
                }
            }
            .NavActive {
                color:#C62F2F;
                &:hover {
                    color:#C62F2F;
                }
                &::before{
                    content: '';
                    width: 57%;
                    height: 3px;
                    background:#C62F2F;
                    position: absolute;
                    bottom: 0
                }
            }

        }
        .searchSong {
            margin-right: 30px;
            .searchBox {
                width: 170px;
                height: 25px;
                border:1px solid $borderColor;
                padding-left: 15px;
                border-radius: 20px;
                box-sizing: border-box;
                font-size: 12px;
                outline: none;
                &::placeholder {
                    font-size: 12px;
                    color:#CCCCCC;
                }
            }
            .icon-sousuo {
                cursor: pointer;
                position: relative;
                left: -25px;
                top: 2px;
                color:#CCCCCC;
            }
        }
    }
    .table {
        width: 100%;
        border-top: 1px solid $borderColor;
        box-sizing: border-box;
        .tableItem {
            box-sizing: border-box;
            padding: 10px 0 10px 10px;
            font-size: 12px;
            color: #666666;
            border-left: 1px solid $borderColor;
        }
        .tControl {
            width: 90px;
        }
        .tMusicTitle {
            width: 33%;
        }
        .tSinger {
            width: 22%;
        }
        .tAlbum {
            width: 22%;
        }
    }
    .sldCommentContainer {
        margin: 20px 30px 40px 30px;
        background: #F0F0F2;
        width: 100%;
        height: 110px;
        .sldComment {
            width: 93.5%;
            background: #FFFFFF;
            margin: 10px;
            border: 1px solid $borderColor;
            height: 50px;
            outline: none;
            &::placeholder {
                font-size: 14px;
                padding-left: 95%;
            }
        }
        .slcConmentIcon {
            display: inline-block;
            i {
                margin-left: 10px;
                &:nth-child(2) {
                    font-size: 18px;
                    position: relative;
                    top: -1px;
                }
                &:nth-child(3) {
                    font-size: 20px;
                    position: relative;
                    top: 1px;
                }
            }
        }
        .sendComment {
            display: inline-block;
            float: right;
            margin-right: 45px;
            background: white;
            border: 1px solid #ccc;
            padding: 5px 8px;
            font-size: 14px;
            border-radius: 5px;
        }
    }
    .comContainer {
        margin-left: 30px;
        .comtitle {
            font-size: 12px;
            margin-bottom: 10px;
        }
        
    }
}
</style>