<template>
  <div class="pageContent">
    <ul class="topNewsList">
      <li v-for="(item,index) in list" :key="index" @click="goDetail(item.Id)">
        <p class="title ellipsis">{{item.Title}}</p>
        <div class="con ellipsis" v-html="item.Memo"></div>
        <img mode="widthFix" :src="item.Pic" class="pic" alt v-if="item.Pic">
      </li>
    </ul>
  </div>
</template>
<script>
import { post,getCurrentPageUrlWithArgs} from "@/utils";
export default {
  data(){
    return {
      pramas:"",//首页头条列表---消息页系统通知
      userId:'',
      token:'',
      curPage:'',
      list:[],//消息列表

    }
  },

  onLoad() {
      this.setBarTitle();
  },
  onShow(){
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.curPage = getCurrentPageUrlWithArgs();
    if(this.$root.$mp.query.url && this.$root.$mp.query.url !==""){
      this.pramas = this.$root.$mp.query.url;
    }else{
      this.pramas = "message";
    }
    
    console.log(this.pramas,"pramas++++++++++++")
    if(this.pramas=="message"){  //消息页系统通知
       console.log('系统通知')
        this.getNotoceList()
    }
    if(this.pramas=="index"){  //首页头条
     console.log('头条')
        this.getNewsList()
    }
    
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "系统通知"
      });
    },
    //获取系统消息列表
    getNotoceList(){
      post('User/GetSysNotice',{
          UserId: this.userId,
          Token: this.token,
          Page:1
      },this.curPage).then(res=>{
        console.log(res,"res")
        this.list = res.data   
      })
    },
    //获取头条
    getNewsList(){
      post('About/AboutList',{PageSize:10}).then(res=>{
        console.log(res)
        this.list = res.data  
      })
    },
    goDetail(id){
        wx.navigateTo({url:'/pages/messages/topNewsDetail/main?url='+this.pramas+'&Id='+id})
    }
  }
};
</script>
<style>

</style>
