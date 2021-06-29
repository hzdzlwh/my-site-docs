<template>
  <div>
    <header class="header">
      <div class="category">
        <a href="/blog">Blog</a>
        <a href="/think">Think</a>
      </div>
    </header>
    <div class="content">
      <Content />
    </div>
  </div>
</template>

<style scoped lang="less">
.header{
  position: relative;
  height: 60px;
  .category{
    position: absolute;
    padding: 0 20px;
    right: 100px;
    a{
      text-decoration: none;
      margin: 3px;
      line-height: 60px;
      font-size: 20px;
      color: rgba(0,0,0,0.4);
      &:hover{
        color: rgba(0,0,0,0.6);
      }
    }
  }
}
.content{
  width: 800px;
  margin: 0 auto;
}
</style>