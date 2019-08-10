<template>
  <div class="pageContent bg_fff mt10">
    <div class="content__hd">
      <h2 class="title">{{newsInfo.Title}}</h2>
      <p class="text_r flex-center">
        <span class="readNum flex-center" >
          <img src="/static/images/icons/time.jpg" alt class="read" style="margin-right:10rpx">{{newsInfo.Addtime}}
        </span>
        <span class="readNum flex-center" >
          <img src="/static/images/icons/read.png" style="margin-right:10rpx" alt class="read">{{newsInfo.Hits}}
        </span>
      </p>
    </div>
    <div class="content__bd">
      <div class="dataHtml" v-html="newsInfo.Content">
        <!-- <p>管是初次创业还是企业发展，写字楼出租都是必不可少的，那么写字楼租赁要注意什么呢？</p>
        <p>首先商圈与地址很重要，这关乎公司的一个发展前景与定位。俗话说物以类聚人以群分嘛。其次要选择一个明亮的大堂，高速快捷的电梯，良好的采光与能让你舒畅的层高；当然你还要考虑到周围的交通是否便利，周围的设施是否完善。</p>
        <p>虽然说地段彰显了写字楼的形象，但写字楼最重要的载体还是建筑，地标性建筑是良好企业形象最佳的阐释。而政府的规划意味着写字楼未来的发展方向，是否可以长久的发展并且能使得写字楼的价值得到提升，这也与政府的规划相关度很高。</p>
        <p>对于写字楼来说，物业管理的服务很重要，选择知名物业对于平时管理上能够省去不少的麻烦。在这个电子信息化的时代，网络与点事办公的两大必需品，所以消防安全也是一个很需要关注的问题，剩下的也就是写字楼租期租赁以及合同的问题了。</p>
        <p>成企业拼租小程序就很好，与其自己花时间的成本去对比，去衡量，还不如把这方面的烦恼交给成成企业拼租小程序，不仅你可以得到更加全面权威的对比，同时也能了解到写字楼租赁的行情，让你省时省力更省心。</p> 
        <img src="/static/images/of/news_b1.jpg" alt>-->
      </div>
    </div>
  </div>
</template>
<script>
//图片在内容里面
import { post,getCurrentPageUrlWithArgs} from "@/utils";
export default {
  data(){
    return {
      newsId:'',
      param:'',
      userId:'',
      token:'',
      curPage:'',
      newsInfo:{}
    }
  },
  onLoad() {
    this.setBarTitle();
  },
  onShow(){
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.curPage = getCurrentPageUrlWithArgs();
    if(this.$root.$mp.query.Id && this.$root.$mp.query.Id !==""){
      this.newsId = this.$root.$mp.query.Id;
      this.getNewsDetail() //获取头条详情
    }
    
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "成成头条"
      });
    },
    //获取头条消息详情
    getNewsDetail(){
      post('About/GetAbout',{
        Id:this.newsId,
        UserId:this.userId,
        Token: this.token
      }).then(res=>{
         if(res.code==0){
           this.newsInfo = res.data
         }
      })
    },
  }
};
</script>
<style lang="scss" scoped>
.title{
  
}
  .pageContent{
  background: #fff;
  font-size: 28rpx;
  line-height: 1.5;
  min-height: 100vh;
  overflow: hidden;
  overflow-y:true;
  width: 100%;
  color: #333;
  position: relative;
}
.readNum{
  margin-right:15rpx;
}
.text_r{
  justify-content:flex-end;
}
</style>