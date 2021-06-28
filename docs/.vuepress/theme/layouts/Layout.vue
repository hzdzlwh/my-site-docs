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
    right: 0;
    a{
      text-decoration: none;
      margin: 3px;
    }
  }
}
.content{
  width: 500px;
  margin: 0 auto;
}
</style>