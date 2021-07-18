<template>
  <div class="menu">
    <ul>
      <li class="item" v-for="item in blogList" :key="item.path">
        <a :href="item.path">{{item.title}}</a>
        <p class="date">{{item.date}} · <span class="readingTime">{{item.readingTime}}</span></p>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      blogList: [
        { path: './clouser', title: '作用域和闭包', date: 'June 10', readingTime: '20min' },
        { path: './this', title: 'this全面解析', date: 'July 13', readingTime: '15min' },
        { path: './vue-reactivity', title: 'Vue响应式原理', date: 'July 17', readingTime: 'xxmin' },
      ]
    }
  }
}
</script>

<style scoped lang="less">
.menu{
  margin-top: 50px;
  .item{
    list-style: none;
    margin: 16px 0;
    a{
      text-decoration: none;
      font-size: 18px;
      color: rgba(0,0,0,0.8);
    }
    .date{
      margin: 0;
      padding: 0;
      font-size: 15px;
      .readingTime{
        color: rgba(0,0,0,0.4);
      }
    }
  }
}
</style>