<template>
  <div class="top-container">
<!--    logo-->
    <div class="logo">
      <ul>
        <li>
          <img src="../assets/GOSICK.jpg" alt="">
          <img src="../assets/徐俊菲.jpg" alt="">
        </li>
      </ul>
      <!--    前进后退按钮-->
      <div class="history-wrapper">
        <span @click="$router.go(-1)" class="iconfont el-icon-arrow-left"></span>
        <span @click="$router.go(1)" class="iconfont el-icon-arrow-right"></span>
      </div>
    </div>
<!--    搜索模块-->
    <div class="search-box">
      <div class="el-input el-input--small el-input--prefix">
        <input
            type="text"
            autocomplete="off"
            placeholder="搜索"
            class="el-input__inner"
            v-model="inputValue"
            @keyup.enter="toResult"
        />
        <span class="el-input__prefix">
          <i class="el-input__icon el-icon-search"></i>
        </span>
      </div>
    </div>
<!--    最右侧三个按钮-->
    <div class="icon-wrapper">
      <span @click="$router.replace('/')" class="iconfont icon-home"></spa
        .n>
      <span class="iconfont icon-sami-select"></span>
      <span class="iconfont icon-full-screen"></span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'top',
  data() {
    return {
      // 输入的内容
      inputValue: ''
    }
  },
  methods: {
    toResult() {
      // 非空判断
      if (this.inputValue == '') {
        // 提示用户
        this.$message.warning('请输入内容')
      }else{
        // 去搜索页 携带数据
        this.$router.push('/result?q='+this.inputValue)
      }
    }
  }
}
</script>

<style scoped></style>
