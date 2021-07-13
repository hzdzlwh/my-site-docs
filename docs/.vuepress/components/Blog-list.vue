<template>
  <ul>
    <li class="item" v-for="item in blogList" :key="item.path">
      <a :href="item.path">{{item.title}}</a>
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      blogList: [
        { path: './clouser', title: '作用域和闭包' },
        { path: './this', title: 'this全面解析' },
      ]
    }
  }
}
</script>

<style scoped lang="less">
  .item{
    list-style: none;
    a{
      text-decoration: none;
      font-size: 20px;
      color: rgba(0,0,0,0.8);
    }
  }
  
</style>