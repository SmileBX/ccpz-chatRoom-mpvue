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
      </div>
      <!-- <p class="text_r flex-center">
        <span class="readNum flex-center" >
          <img src="/static/images/icons/time.jpg" alt class="read" style="margin-right:10rpx">{{newsInfo.Addtime}}
        </span>
        <span class="readNum flex-center" >
          <img src="/static/images/icons/read.png" style="margin-right:10rpx" alt class="read">{{newsInfo.Hits}}
        </span>
      </p> -->
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
    this.newsInfo={};
    if(this.$root.$mp.query.Id && this.$root.$mp.query.Id !==""){
      this.newsId = this.$root.$mp.query.Id;
    }
    if(this.$root.$mp.query.url && this.$root.$mp.query.url!==""){
       this.param = this.$root.$mp.query.url;
    }
    if(this.param == 'message'){ //获取通知详情
      console.log('通知')
        this.getNoticeDetail()
    }else{
         console.log('头条')
        this.getNewsDetail() //获取头条详情
    }
  },
  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "系统通知"
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
           res.data.Content = res.data.Content.replace(/\<img/gi, '<img style="max-width:100%;height:auto" ')
           this.newsInfo = res.data
         }
      })
    },
    //通知详情
    getNoticeDetail(){
      post('User/ReadSysNotice',{
          UserId: this.userId,
          Token: this.token,
          NoticeId:this.newsId  
      },this.curPage).then(res=>{
        if(res.code==0){
           this.newsInfo = {
             Title:res.data.Title,
             PubTime:res.data.PubTime.split(" ")[0].split("-").join("."),
             Content:res.data.Memo.replace(/\<img/gi, '<img style="max-width:100%;height:auto" ')
           }
         }
        console.log(this.newsInfo.Content,"通知详情")
      })
    }
  }
};
</script>
<style lang="scss" scoped>
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