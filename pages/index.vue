<template>
  <el-container class="container">
    <el-header class="header">
      <el-input
        placeholder="请输入歌曲"
        v-model="seachVal"
        class="search"
        @change="handleClick"
      >
        <el-button slot="append" icon="el-icon-search"
        v-loading.fullscreen.lock="loading"
          @click="handleClick"
        ></el-button>
      </el-input>
    </el-header>
    <el-main class="main">
      <ul v-infinite-scroll="load" infinite-scroll-delay="300" infinite-scroll-distance="0" :infinite-scroll-disabled="!disabled">
        <li v-for="item in result.songs" :key="item.id">{{item.name}} -- {{item.ar[0].name}}</li>
      </ul>
    </el-main>
  </el-container>
</template>

<script lang="ts">
import axios from "axios"
export default {
  data(){
    return {
      seachVal: "",
      result: {
        hasMore: true,
        songCount: 0,
        songs:[]
      },
      page: 0,
      pageNum: 80,
      loading: false,
    }
  },
  computed: {
    disabled(): Boolean{
      return this.loading || this.result.hasMore
    }
  },
  methods: {
    handleClick(){
      this.seachVal = this.seachVal.trim()
      this.page = 0
      this.result.songCount = 0
      this.result.songs = []
      this.load()
    },
    async load(): Promise<void>{
      if(!this.seachVal.trim()){
        return;
      }
      this.loading = true
     let result: any = await axios({
        url: "/api/cloudsearch",
        method: "GET",
        params: {
          keywords: this.seachVal,
          limit: this.pageNum * ++this.page
        }
      })
      this.loading = false
      if(result.data.code == 200){
        this.result = {
          ...this.result,
          ...result.data.result
        }
        console.log(this.result)
      }else if(result.data.code == 400){
        this.result.hasMore = false
        this.$message({
          message: "没有更多歌曲了",
          type: "info"
        })
      }
    }
  }
};
</script>

<style scoped>

.container{
  width: 100%;
  height: 100%;
  overflow: auto;
}
.header {
  position: relative;
}
.search{
  width: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateY(-50%) translateX(-50%);
}
.main {
  width: 72%;
  margin: 0 auto;
}
ul,li {
  list-style: none;
}
</style>
