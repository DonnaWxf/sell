<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab">
      <div class="tab-item">
        <router-link to='/goods' class="goods">商品</router-link>
        </div>
      <div class="tab-item">
         <!-- <a v-link="{path:'/ratings'}">评论</a> -->
        <router-link to='/ratings'>评论</router-link>
      </div>
      <div class="tab-item">
         <!-- <a v-link="{path:'/seller'}">商家</a> -->
        <router-link to='/seller'>商家</router-link>
      </div>
    </div>
    <div class="content">
      <router-view></router-view>
    </div>
  </div>
</template>

<script>
import header from '@/components/header/header.vue'
const ERR_OK = 0
export default {
  name: 'App',
  data () {
    return {
      seller: ''
    }
  },
  created () {
    this.$http.get('/api/seller').then((res) => {
      res = res.body;
      // res = res.json();
      if (res.errno === ERR_OK) {
        this.seller = res.data;
        console.log('res', this.seller)
      }
    })
  },
  components: {
    'v-header': header
  }
}

</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.tab{
  display: flex;
  width: 100%;
  height: 40px;
  line-height: 40px;
  border-bottom:1px solid rgba(7,17,27,0.1);
  .tab-item{
    flex: 1;
    text-align: center;
    font-size: 14px;
    a{
      color:rgb(77, 85, 93);
    }
    .router-link-active{
      color:rgb(240, 20, 20);
    }
  }
}
</style>
