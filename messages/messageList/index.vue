<template>
  <div class="pageContent">
    <ul class="message">
      <li v-for="(item,index) in list" :key="index" @click="goDetail(item.Id)">
        <div class="time">{{item.PubTime}}</div>
        <div class="list">
          <img src="/static/images/icons/notRead.png" alt="" class="notRead" v-if="item.IsRead!==0">
          <p class="title ellipsis" >{{item.Title}}</p>
          <img mode="widthFix" :src="item.Pic" class="pic" alt v-if="item.Pic">
          <!-- <div class="con ellipsis content" v-html="item.Memo"></div> -->
          <div class="con ellipsis content" >{{item.Memo}}</div>
          <div class="detail">查看详情<span>></span></div>
        </div>
      </li>
    </ul>
      <!-- 暂无数据等提示 -->
      <div class="noData center" style="padding:60rpx 30rpx;" v-if="list.length<1&& page===1">暂无数据</div>
      <div
        class="noData center"
        style="margin-top:0;line-height:80rpx;"
        v-if="hasData&& page!==1"
      >我也是有底线的!</div>
      <!-- 暂无数据等提示  end -->
  </div>
</template>
<script>
import { post,getCurrentPageUrlWithArgs} from "@/utils";
export default {
  data(){
    return {
      userId:'',
      token:'',
      curPage:'',
      page:1,
      pageSize:12,
      list:[],//消息列表
      hasData:false
    }
  },

  onLoad() {
      this.setBarTitle();
  },
  onShow(){
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.curPage = getCurrentPageUrlWithArgs();
    this.init()
    
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "系统通知"
      });
    },
    init(){
    this.hasData =false;
    this.page=1;
    this.list=[];
    this.getNotoceList()
    },
    //获取系统消息列表
    getNotoceList(){
      post('User/GetSysNotice',{
          UserId: this.userId,
          Token: this.token,
          Page:this.page,
          PageSize:this.pageSize
      },this.curPage).then(res=>{
        // res.data.Memo = res.data.Memo.replace(/\<img/gi, '<img style="max-width:100%;height:auto" ');
        res.data.map(item=>{
              // item.Memo = item.Memo.replace(/\<img.*?\"\>/gi,'')
              item.Memo = item.Memo.replace(/\<.*?\>/gi,'')
        })
        this.list = this.list.concat(res.data)   
        res.data.length!==this.pageSize&&(this.hasData = true)
      })
    },
    goDetail(id){
        wx.navigateTo({url:'/pages/messages/messageDetail/main?Id='+id})
    }
  },
  // 上拉加载
  onReachBottom() {
    if(this.hasData)return false;
    this.page+=1
    this.getNotoceList();
  },
  //下拉刷新
  onPullDownRefresh() {
    this.init()
    wx.stopPullDownRefresh();
  },
};
</script>
<style  lang="scss" scoped>
  .notRead{
    position:absolute;
    top:0;
    right:0;
    width:80rpx!important;
    height:80rpx!important;
    z-index:10;
  }
.message{
  padding:0 20rpx;
  .time{
    color:#999;
    line-height:90rpx;
    text-align:center;
    font-size:22rpx;
  }
.list{
  background:#fff;
  display:flex;
  flex-flow:column nowrap;
  padding:0 30rpx;
  border-radius:8rpx;
  position:relative;
  .title{
    font-size:32rpx;
    font-weight:600;
      line-height:90rpx;
  }
    img{
      width:100%;
    }
    .content{
      color:#999;
      line-height:80rpx;
      white-space:nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  .detail{
    display:flex;
    justify-content:space-between;
    line-height:70rpx;
    border-top:1rpx #e8e8e8 solid;
  }
}
}
.c999{
  color:#999;
}
</style>
