<template>
	<div class="wrap">
    <div class="top">
      <h1>403</h1>
    </div>
    <div class="bottom">
      <h3>Sorry, you are not authorized to access this page</h3>
      <h3>抱歉，您无权访问此页面</h3>
      <el-button type="default" @click="back">点此返回</el-button>
    </div>
  </div>
</template>
<script>
export default {
  methods: {
    back () {
      this.$router.back()
    }
  }
}
</script>
<style scoped lang="scss">
  .wrap {
    position: fixed;
    width: 100%;
    height: 100%;
    color: #676a6c;
    text-align: center;
    background: #fff;
    .top {
      height: 70%;
      background: gray;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .top h1 {
      font-size: 200px;
      color: #fff;
    }
    .bottom {
      padding-top: 30px;
      h3 {
        margin: 15px 0;
      }
    }
  }
</style>