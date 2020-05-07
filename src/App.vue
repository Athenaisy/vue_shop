<template>
  <div id="app">
   <!--路由占位符 -->
   <router-view></router-view>
  </div>
</template>

<script>


export default {
  name: 'app',

};
</script>

<style lang="less">
html,
body,
#app{
  margin: 0;
  padding: 0;
  width:100%;
  height: 100%;
}
</style>
