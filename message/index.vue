<template>
  <div class="page bg_fff borderTop" v-if="isData">
    <!--顶部输入框-->
    <div class="bg_fff flex flexAlignCenter pall">
      <img src="/static/images/icons/search.png" alt class="icon_search">
      <input
        type="text"
        placeholder="请输入要搜索的联系人名字/最新消息"
        class="flex1 bg_fff"
        v-model="inputName"
        @input="changeSearch"
      >
    </div>
    <div class="slidebg"></div>
    <!--通知-->
    <div>
            <!-- <van-swipe-cell :right-width="65" > -->
              <div class="padwid">
                <div class="flex avatarhead flexAlignCenter boxSize" @click="toNociceList">
                    <div class="circlePosition">
                        <div class="avatarbox">
                              <img src="/static/images/icons/default.jpg" alt="" class="avatar">
                        </div>
                        <span class="circleNum msgnum" v-if="messageInfo.SysNotice.Count>0">{{messageInfo.SysNotice.Count}}</span>
                    </div>
                      <div class="flex flexColumn flex1">
                          <div class="flex">
                              <span class="flex1 font32">系统通知</span>
                              <span class="font_four">{{messageInfo.SysNotice.PubTime}}</span>
                          </div>
                          <div class="fontColor text_space">{{messageInfo.SysNotice.Title}}</div>
                      </div>
                </div>
                </div>
                <!-- <span slot="right" class="informRead">已读</span> -->
            <!-- </van-swipe-cell> -->
              <div class="padwid">
        <div class="flex flexAlignCenter p1 boxSize" @click="seePermistion">
          
            <div class="avatarbox">
                  <img v-if="messageInfo.friend_req.Headimgurl" :src="messageInfo.friend_req.Headimgurl" alt="" class="avatar">
                  <img v-else src="/static/images/icons/pinzu.jpg" alt="" class="avatar">
            </div>
            <div class="flex1 flex circlePosition">
                <span class="flex1 font32">好友请求</span>
                <span class="circleNum onlynum" v-if="messageInfo.friend_req.Count>0">{{messageInfo.friend_req.Count}}</span>
            </div>
            </div>
        </div>
    </div>
    <div class="slidebg"></div>
    <!--消息列表-->
    <div class="padwid" v-if="list.length>0">
        <div class="flex avatarhead flexAlignCenter boxSize p2" 
          @click="goChat(item.FriendId,item.TempId)" v-for="(item,index) in list" :key="index">
            <div class="circlePosition">
                <div class="avatarbox">
                  <img :src="item.Headimgurl" alt="" class="avatar">
                </div>
                <span class="circleNum msgnum" v-if="item.Count>0">{{item.Count }}</span>
            </div>
            <div class="flex flexColumn flex1">
                <div class="flex">
                    <span class="flex1 font32">{{item.NickName}}</span>
                    <span class="font_four">{{item.AddTime}}</span>
                </div>
                <div class="fontColor">{{item.Info}}</div>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import { post,getCurrentPageUrlWithArgs,getNewMsgDot} from "@/utils";
export default {
  data () {
    return {
      userId:'',
      token:'',
      curPage:'',
      isData:false,//是否开始渲染
      messageInfo:{},
      inputName:'',
      list:[], //联系人.
    }
  },
   onLoad() {
    this.setBarTitle();
    this.curPage = getCurrentPageUrlWithArgs();
  },
  onShow(){
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    console.log(this.curPage)
    this.getMessage()
    getNewMsgDot();
  },

  components: {
  },

  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "消息"
      });
    },
    //获取消息
    getMessage(){
      post('User/GetMessageList',{
          UserId: this.userId,
          Token: this.token
      },this.curPage).then(res=>{
          console.log(res)
          this.messageInfo = res.data;
          this.list = res.data.friend_new;
          this.isData = true;
      })
    },
    //进入聊天室
    goChat(fid,TempId){
        wx.navigateTo({url: `/pages/messages/chatRoom/main?FriendId=${fid}&TempId=${TempId}`})
    },
    //查看系统通知列表
    toNociceList(){
      // wx.navigateTo({url:'/pages/messages/messageList/main?type=2'})
      wx.navigateTo({url:'/pages/messages/headLineList/main?type=2'})
    },
    //查看好友请求列表
     seePermistion(){
        wx.navigateTo({url: '/pages/messages/newPromise/main'})
    },
    // 搜索最近联系人
    changeSearch(){
      console.log(this.inputName)
      const value = this.inputName;
      this.list=[];
      this.messageInfo.friend_new.map((item,index)=>{
        const i = item.NickName.indexOf(value)
        const infoIndex = item.Info.indexOf(value)
        if(i!==-1||infoIndex!==-1){
          this.list.push(item);
        }
      })
      this.list.length===0&&(this.list = this.messageInfo.friend_new)
    }
  },
  //下拉刷新
  onPullDownRefresh() {
    wx.stopPullDownRefresh();
    this.getMessage()
    getNewMsgDot();
  },
  created () {
    // let app = getApp()
  }
}
</script>
<style lang="scss" scoped>
.informRead{
  background:#ff952e;
  color:#fff;
  height:170rpx;
  line-height:170rpx;
  width:130rpx;
  display:inline-block;
  text-align:center;
}
</style>

