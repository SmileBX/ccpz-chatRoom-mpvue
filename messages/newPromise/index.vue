<template>
  <div class="page bg_fff borderTop">
    <!--通知列表-->
    <div class="">
      <div class="listBox" v-for="(noticeItem,noticeIndex) in noticeList" :key="noticeIndex">
        <div class="timeLine">{{noticeItem.name}}</div>
        <div class="flex flexAlignCenter boxSize p30 listItem" v-for="(item,index) in noticeItem.list" :key="index">
            <div class="avatarbox">
                <img :src="item.Headimgurl" alt="" class="avatar">
            </div>
            <div class="flex flex1 flexAlignCenter">
                <div class="flex flex1 flexColumn">
                    <span class="font32">{{item.NickName}}</span>
                    <span class="font_four">{{item.AuthInfo}}</span>
                </div>
                <div class="btnadd" v-if="item.Status==1">已添加</div>
                <div class="btnadd" v-if="item.Status==2">已忽略</div>
                <div class="ignoreBtn flex flexAlignCenter" v-if="item.Status==0">
                    <span class="ignore boxSize" @click="dealPermission(item.Id,2)">忽略</span>
                    <span class="pass boxSize" @click="dealPermission(item.Id,1)">通过</span>
                </div>
            </div>
        </div>
      </div>
    </div>
    <p
      style="text-align:center;font-size:30rpx;color:#666;padding:120rpx 20rpx 80rpx;"
      v-if="hasData===false"
    >暂无数据</p>
    <p
      class="ovedMsg"
      v-if="isOved && page>1"
      style="text-align:center;padding:20rpx;font-size:26rpx;color:#666;"
    >我也是有底线的</p>
  </div>
</template>

<script>
//Status, --0:未处理 1:已添加 2:已忽略 3:已拒绝 4:等待验证
import { post,getCurrentPageUrlWithArgs} from "@/utils";
export default {
  data () {
    return {
      showbtn:false,//是否已添加
      showbtn2:false,//是否已忽略
      userId:'',
      token:'',
      curPage:'',
      Page:1,
      PageSize:100,
      noticeList:[],
      hasData:false,//是否有数据，
      isOved:false,//是否加载完了数据

    }
  },
   onLoad() {
    this.setBarTitle();
  },
  onShow(){
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.curPage = getCurrentPageUrlWithArgs();
    this.initData()
    this.getNewPromise()
  },
  components: {
  },

  methods: {
    initData(){
        this.Page=1
        this.noticeList=[]
        this.isOved = false;
        this.hasData = "";

    },
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "新朋友"
      });
    },
    //获取好友请求列表
    getNewPromise(){
      post('User/Getfriend_req',{
          UserId: this.userId,
          Token: this.token,
          Page:this.Page,
          PageSize:this.PageSize
      },this.curPage).then(res=>{
        console.log(res,"getNewPromise")
        if(res.data.length===0){
            this.hasData = false
        }else{
            this.hasData = true
            let nameArr = []
            res.data.map(item=>{
              nameArr.push(item.AddTime)
            })
            // arr = Array.from(new Set(arr))//数组去重
            nameArr = [...new Set(nameArr)]//数组去重
            let listArr=[]
            nameArr.map(item=>{
              listArr.push({
                name:item,
                list:[]
              })
            })
            res.data.map(item=>{
              listArr.map(listItem=>{
                if(item.AddTime===listItem.name){
                  listItem.list.push(item)
                }
              })
            })
            console.log(listArr,'arr')
            this.noticeList = listArr
        }
      })
    },
    //处理好有请求
    dealPermission(id,ReqStatus){
      //1:添加 2:忽略
        post('User/Confriend_req',{
          UserId: this.userId,
          Token: this.token,
          ReqId:id,
          ReqStatus:ReqStatus
        },this.curPage).then(res=>{
            console.log(res,"处理好有请求")
            if(res.code==0){
                 wx.showToast({
                  title:'操作成功！',
                  duration:2000
                })
                // if(ReqStatus==1){
                //     this.$set(this.noticeList[index],'Status',1)
                // }else if(ReqStatus==2){
                //     this.$set(this.noticeList[index],'Status',2)
                // }
                this.initData();
                this.getNewPromise()
            }
           
        })
    }
  },

  created () {
    // let app = getApp()
  },
  onPullDownRefresh() {
    wx.stopPullDownRefresh();
    // 下拉刷新
     this.initData();
     this.getNewPromise()
  },
  // onReachBottom() {
  //   //上拉加载
  //   if (!this.isOved) {
  //     this.page++;
  //     this.getNewPromise()
  //   }
  // }
}
</script>
<style lang="scss" scoped>
.p30{
  padding:30rpx;
}
  .timeLine{
    font-size:24rpx;
    color:#999;
    line-height:60rpx;
    background:#f2f2f2;
    padding:0 30rpx;
  }
  .listItem{
    border-bottom:1rpx #ececec solid;
  }
</style>