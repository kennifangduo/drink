<template>
  <div class="search">
    <van-nav-bar left-text="返回" left-arrow fixed @click-left="back">
      <template #right>
        <div class="home-search">
          <van-search
            v-model="name"
            ref="search(name)"
            show-action
            @search="search"
            placeholder="输入商品名称"
          />
        </div>
      </template>
    </van-nav-bar>
    <div class="inputTextBox">
      <p>历史记录</p>
      <div class="text">
        <span v-for="(val, index) in historyData.code" :key="index" @click="getHistory(val)">{{val}}</span>
      </div>
      <van-icon name="delete" @click="removeHistory"/>
    </div>

     <BgBox>
      <div class="clearfix">
        <div class="fl like-item" v-for="(item, index) in products" :key="index" @click="goDetail(item.pid)">
          <ProductItem :item="item"></ProductItem>
        </div>
      </div>
    </BgBox>

  </div>
</template>

<script>
import BgBox from "../components/BgBox.vue";
import ProductItem from "../components/ProductItem.vue";
export default {
  name: "Search",

  components: {
    BgBox,
    ProductItem,
  },

  data() {
    return {
      //搜索商品关键字
      name: "",

      //搜索商品数据
      products: [],

      // 是否显示历史记录

      // 历史记录
      historyData:{code:[]}
    };
  },

  created() {
    
    this.$nextTick(() => {
      
      //获取搜索框
      let searchIpt = this.$refs["search(name)"]["querySelector"]('[type="search"]');
      

      //获取焦点
      searchIpt.focus();ses

    });
    
    if(localStorage.getItem("historyData")){
      this.historyData = JSON.parse(localStorage.getItem("historyData"));
    }
    if(sessionStorage.getItem("backData")){
      this.search(sessionStorage.getItem("backData"));
      this.name = sessionStorage.getItem("backData");
    }
  },

  methods: {
    back() {
      sessionStorage.removeItem("backData");
      this.$router.go(-1);
    },

    //搜索
    search(name) {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });

      this.axios({
        method: "GET",
        url: "/search",
        params: {
          appkey: this.appkey,
          name: name,
        },
      })
        .then((result) => {
          this.$toast.clear();
          

          if (result.data.code == "Q001" && name!="") {
            this.products = result.data.result;
            this.getHistoryData();
            sessionStorage.setItem("backData",name);
          }
        })
        .catch((err) => {
          this.$toast.clear();
          
        });
    },

    // 获取历史
    getHistory(val){
      this.$el.firstChild.firstChild.lastChild.firstChild.firstChild.firstChild.lastChild.lastChild.firstChild.firstChild.value = val;
      this.name=val;
      this.search(val);
    },

    // 初始化历史
    getHistoryData(){
      if(localStorage.getItem("historyData")){
        this.historyData = JSON.parse(localStorage.getItem("historyData"))
      }
      let val = this.$el.firstChild.firstChild.lastChild.firstChild.firstChild.firstChild.lastChild.lastChild.firstChild.firstChild.value;
      if(this.historyData.code.indexOf(val)<0){
        this.historyData.code.push(val);
        localStorage.setItem("historyData", JSON.stringify(this.historyData))
      }
    },

    // 删除历史
    removeHistory(){
      localStorage.removeItem("historyData");
      this.historyData = {code:[]};
      this.$el.childNodes[1].childNodes[1].innerHTML = ""
      this.bool = false;
    },

     //查看商品详情
    goDetail(pid) {
      
      this.$router.push({name: 'Detail', params: {pid}});
    },
  },
};
</script>

<style lang="less" scoped>
.search {
  padding-top: 46px;

  /deep/ .van-nav-bar__content{
    background-color: #f5f5f5;
  }

  /deep/ .van-nav-bar .van-icon {
    color:  #ffa500;
  }

  /deep/ .van-nav-bar__text {
    color: #ffa500;
  }

  /deep/ .van-nav-bar__right {
    width: calc(~"100% - 110px");
  }

  /deep/ .home-search {
    width: 100%;
  }
  /deep/ .home-search .van-search {
    padding: 0;
    border-radius: 17px;
    overflow: hidden;
  }

  /deep/ .home-search .van-icon {
    color: #ffa500;
  }

  /deep/ .van-search__content{
    background-color: #ffffff;
  }

  .like-item{
    width: calc(~"100% / 3 - 10px + 10px / 3");
    margin-right: 10px;
    margin-bottom: 10px;
    &:nth-child(3n){
      margin-right: 0;
    }
  }
  .inputTextBox{
    position: absolute;
    top: 51px;
    left: 10px;
    z-index: 99999;
    width: 95%;
    height: 70px;
    box-sizing: border-box;
    overflow: hidden;
    border-radius: 5px;
    padding: 8px 15px 10px 0px;
    background-color: rgba(0, 0, 0, .6);
    color: #fff;
    font-size: 14px;
    p{
      margin: 0;
      padding: 0;
      margin-bottom: 5px;
    }
    i{
      position: absolute;
      bottom: 5px;
      right: 4px;
      font-size: 16px;
      color: #cccccc;
    }
    span{
      overflow: hidden;
      display: inline-block;
      margin-left: 10px;
    }
  }
}
</style>