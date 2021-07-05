<template>
  <div class="result-container">
    <div class="title-wrap">
      <!--      顶部标题-->
      <h2 class="title">{{ $route.query.q }}</h2>
      <span class="sub-title">找到{{ count }}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <!--歌曲-->
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
          <th></th>
          <th>音乐标题</th>
          <th>歌手</th>
          <th>专辑</th>
          <th>时长</th>
          </thead>
          <tbody>
          <tr v-for="(item,index) in songList" :key="index" class="el-table__row" @dblclick="playMusic(item.id)">
            <td>{{ index + 1 }}</td>
            <td>
              <div class="song-wrap">
                <div class="name-wrap">
                  <!--歌名-->
                  <span>{{ item.name }}</span>
                  <!-- mv的图标 -->
                  <span v-if="item.mvid != 0" class="iconfont icon-mv" @click="toMV(item.mvid)"></span>
                </div>
                <!--二级标题-->
                <span v-if="item.alias.length!=0">{{ item.alias[0] }}</span>
              </div>
            </td>
            <td>{{ item.artists[0].name }}</td>
            <td>{{ item.album.name }}</td>
            <td>{{ item.duration }}</td>
          </tr>
          </tbody>
        </table>
      </el-tab-pane>
      <!--歌单-->
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div v-for="(item,index) in playlists" :key="index" class="item" @click="toPlaylist(item.id)">
            <div class="img-wrap">
              <!--播放量-->
              <div class="num-wrap">
                播放量:
                <span class="num">{{ item.playCount }}</span>
              </div>
              <!--封面-->
              <img :src="item.coverImgUrl" alt=""/>
              <span class="iconfont icon-play"></span>
            </div>
            <!--歌单名-->
            <p class="name">{{ item.name }}</p>
          </div>
        </div>
      </el-tab-pane>
      <!--mv-->
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div v-for="(item,index) in mv" :key="index" class="item" @click="toMV(item.id)">
            <div class="img-wrap">
              <!--封面-->
              <img :src="item.cover" alt=""/>
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <!--播放次数-->
                <div class="num">{{ item.playCount }}</div>
              </div>
              <!--持续时间-->
              <span class="time">{{ item.duration }}</span>
            </div>
            <div class="info-wrap">
              <!--mv名字-->
              <div class="name">{{ item.name }}</div>
              <!--歌手-->
              <div class="singer">{{ item.artistName }}</div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
    <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="limit">
    </el-pagination>
  </div>
</template>

<script>
//导入axios
import axios from "axios";

export default {
  name: 'result',
  data() {
    return {
      // tab切换时 会改变的数据
      activeIndex: 'songs',
      // 保存 查询歌曲
      songList: [],
      // 保存歌单的字段
      playlists: [],
      // 保存mv的字段
      mv: [],
      // 搜索结果的总数
      count: 0,
      //页码
      page: 1,
      //页容量
      limit: 10,
      //总条数
      total: 0
    };
  },
  // 生命周期钩子 回调函数
  created() {
    axios({
      url: 'https://autumnfish.cn/search',
      method: 'get',
      params: {
        keywords: this.$route.query.q,
        type: 1,
        // 获取的数据量
        limit: 12
      }
    }).then(res => {
      // console.log(res)
      this.total = res.data.result.songCount
      this.songList = res.data.result.songs
      // 计算歌曲时间
      for (let i = 0; i < this.songList.length; i++) {
        let min = parseInt(this.songList[i].duration / 1000 / 60)
        let sec = parseInt((this.songList[i].duration / 1000) % 60)
        if (min < 10) {
          min = '0' + min
        }
        if (sec < 10) {
          sec = '0' + sec
        }
        // console.log(min + '|' + sec)
        this.songList[i].duration = min + ':' + sec
      }
      // 保存总数
      this.count = res.data.result.songCount
    })
  },
  //侦听器
  watch: {
    activeIndex() {
      this.page = 1
      // songs 歌曲
      // lists 歌单
      // mv    mv
      let type = 1

      // 获取个数
      let limit = 12
      switch (this.activeIndex) {
        case "songs":
          type = 1
          limit = 12
          break
        case "lists":
          type = 1000
          limit = 10
          break
        case "mv":
          type = 1004
          limit = 8
          break
        default:
          break
      }
      // 获取个数
      axios({
        url: 'https://autumnfish.cn/search',
        method: 'get',
        params: {
          keywords: this.$route.query.q,
          type,// type:type,
          // 获取的数据量
          limit,// limit:limit
          offset: (this.page - 1) * limit,
        }
      }).then(res => {
        // console.log(res)
        //获取歌曲
        if (type == 1) {
          this.total = res.data.result.songCount
          this.songList = res.data.result.songs
          // 计算歌曲时间
          for (let i = 0; i < this.songList.length; i++) {
            let min = parseInt(this.songList[i].duration / 1000 / 60)
            let sec = parseInt((this.songList[i].duration / 1000) % 60)
            if (min < 10) {
              min = '0' + min
            }
            if (sec < 10) {
              sec = '0' + sec
            }
            // console.log(min + '|' + sec)
            this.songList[i].duration = min + ':' + sec
          }
          // 保存总数
          this.count = res.data.result.songCount
        }
        // 获取 歌单
        else if (type == 1000) {
          this.total = res.data.result.playlistCount
          // 歌单的逻辑
          this.playlists = res.data.result.playlists
          // 总数
          this.count = res.data.result.playlistCount
          // 处理 播放次数
          for (let i = 0; i < this.playlists.length; i++) {
            if (this.playlists[i].playCount > 100000) {
              this.playlists[i].playCount =
                  parseInt(this.playlists[i].playCount / 10000) + '万'
            }
          }
        } else {
          this.total = res.data.result.mvCount
          // 保存mv数据
          this.mv = res.data.result.mvs
          // 总数
          this.count = res.data.result.mvCount
          // 处理数据
          for (let i = 0; i < this.mv.length; i++) {
            // 时间
            let min = parseInt(this.mv[i].duration / 1000 / 60)
            let sec = parseInt((this.mv[i].duration / 1000) % 60)
            if (min < 10) {
              min = '0' + min
            }
            if (sec < 10) {
              sec = '0' + sec
            }
            this.mv[i].duration = min + ':' + sec

            // 播放次数
            if (this.mv[i].playCount > 100000) {
              this.mv[i].playCount =
                  parseInt(this.mv[i].playCount / 10000) + '万'
            }
          }
        }
        // this.songList = res.data.result.songs
        // // 计算歌曲时间
        // for (let i = 0; i < this.songList.length; i++) {
        //   let min = parseInt(this.songList[i].duration / 1000 / 60)
        //   let sec = parseInt((this.songList[i].duration / 1000) % 60)
        //   if (min < 10) {
        //     min = '0' + min
        //   }
        //   if (sec < 10) {
        //     sec = '0' + sec
        //   }
        //   // console.log(min + '|' + sec)
        //   this.songList[i].duration = min + ':' + sec
        // }
        // // 保存总数
        // this.count = res.data.result.songCount
      })
    }
  },
  //方法
  methods: {
    searchResult() {
      // songs 歌曲
      // lists 歌单
      // mv    mv
      let type = 1

      // 获取个数
      let limit = 12
      switch (this.activeIndex) {
        case "songs":
          type = 1
          limit = 12
          break
        case "lists":
          type = 1000
          limit = 10
          break
        case "mv":
          type = 1004
          limit = 8
          break
        default:
          break
      }
      // 获取个数
      axios({
        url: 'https://autumnfish.cn/search',
        method: 'get',
        params: {
          keywords: this.$route.query.q,
          type,// type:type,
          // 获取的数据量
          limit,// limit:limit
          offset: (this.page - 1) * limit,
        }
      }).then(res => {
        // console.log(res)
        //获取歌曲
        if (type == 1) {
          this.total = res.data.result.songCount
          this.songList = res.data.result.songs
          // 计算歌曲时间
          for (let i = 0; i < this.songList.length; i++) {
            let min = parseInt(this.songList[i].duration / 1000 / 60)
            let sec = parseInt((this.songList[i].duration / 1000) % 60)
            if (min < 10) {
              min = '0' + min
            }
            if (sec < 10) {
              sec = '0' + sec
            }
            // console.log(min + '|' + sec)
            this.songList[i].duration = min + ':' + sec
          }
          // 保存总数
          this.count = res.data.result.songCount
        }
        // 获取 歌单
        else if (type == 1000) {
          this.total = res.data.result.playlistCount
          // 歌单的逻辑
          this.playlists = res.data.result.playlists
          // 总数
          this.count = res.data.result.playlistCount
          // 处理 播放次数
          for (let i = 0; i < this.playlists.length; i++) {
            if (this.playlists[i].playCount > 100000) {
              this.playlists[i].playCount =
                  parseInt(this.playlists[i].playCount / 10000) + '万'
            }
          }
        } else {
          this.total = res.data.result.mvCount
          // 保存mv数据
          this.mv = res.data.result.mvs
          // 总数
          this.count = res.data.result.mvCount
          // 处理数据
          for (let i = 0; i < this.mv.length; i++) {
            // 时间
            let min = parseInt(this.mv[i].duration / 1000 / 60)
            let sec = parseInt((this.mv[i].duration / 1000) % 60)
            if (min < 10) {
              min = '0' + min
            }
            if (sec < 10) {
              sec = '0' + sec
            }
            this.mv[i].duration = min + ':' + sec

            // 播放次数
            if (this.mv[i].playCount > 100000) {
              this.mv[i].playCount =
                  parseInt(this.mv[i].playCount / 10000) + '万'
            }
          }
        }
        // this.songList = res.data.result.songs
        // // 计算歌曲时间
        // for (let i = 0; i < this.songList.length; i++) {
        //   let min = parseInt(this.songList[i].duration / 1000 / 60)
        //   let sec = parseInt((this.songList[i].duration / 1000) % 60)
        //   if (min < 10) {
        //     min = '0' + min
        //   }
        //   if (sec < 10) {
        //     sec = '0' + sec
        //   }
        //   // console.log(min + '|' + sec)
        //   this.songList[i].duration = min + ':' + sec
        // }
        // // 保存总数
        // this.count = res.data.result.songCount
      })
    },
    // 去mv详情页
    toMV(id) {
      this.$router.push(`/mv?q=${id}`)
    },
    //跳转到歌单详情页面
    toPlaylist(id) {
      //跳转并携带数据
      this.$router.push(`/playlist?q=${id}`)
    },
    //播放音乐
    playMusic(id) {
      axios({
        url: 'https://autumnfish.cn/song/url',
        method: 'get',
        params: {
          id // id:id
        }
      }).then(res => {
        // console.log(res)
        let url = res.data.data[0].url
        // console.log(this.$parent.musicUrl)
        // 设置给父组件的 播放地址
        this.$parent.musicUrl = url
      })
    },
    //页码改变
    handleCurrentChange(val) {
      //保存页码
      this.page = val;
      //重新获取数据
      this.searchResult();
    },
  }
};
</script>

<style>

</style>
