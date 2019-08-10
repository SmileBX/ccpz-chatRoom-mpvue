<template>
  <div class="page">
      <div class="head bg_fff ">
        <!--头像部分-->
          <div class="flex avatarhead">
              <div class="avatarbox">
                  <img :src="userInfo.Avatar" alt="" class="avatar">
              </div>
              <div class="flex1">
                  <p>{{userInfo.Name}}</p>
                  <p class="font_four" v-if="userInfo.Area">{{userInfo.Area}}</p>
              </div>
          </div>
          <!--填写验证信息-->
          <div class="vertical flex flexColumn">
              <p class="font_four">填写验证信息</p>
              <input type="text" v-model.trim="verMsg" placeholder="我是..." class="flex1 yanz">
          </div>
      </div>
      <!-- <div class="setclass">设置备注与分组</div>
      <div class="bg_fff group boxSize">
          <div class="flex flexAlignCenter slide boxSize sign">
              <p>备注</p>
              <input type="text"  v-model="specName" class="flex1" placeholder="请输入备注">
          </div>
          <div class="flex flexAlignCenter slide boxSize">
              <p>分组</p>
              <p class="flex flex1" @tap="choseTeam">
                  <input type="text" disabled v-model="specPlace" placeholder="请选择分组名称" class="flex1">
                  <span class="icon-arrow arrow-right"></span>
              </p>
          </div>
      </div> -->
      <!--加好友-->
      <div class="btnSub addtop" @click="addFriend">加好友</div>
  </div>
</template>

<script>
import {post,getLocation} from '@/utils'
export default {
  data () {
    return {
      userId:'',
      token:'',
      FriendId:'16',
      userInfo:{},
      lat:0,
      lng:0,
      verMsg:'',
      specName:'',
      specPlace:'',
    }
  },
  onLoad() {
    this.setBarTitle();
    
  },
  onShow(){
    this.userId = wx.getStorageSync('userId')
    this.token = wx.getStorageSync('token')
    if(this.$root.$mp.query.FriendId){
      this.FriendId = this.$root.$mp.query.FriendId
    }
    if(this.$root.$mp.query.groupName){
      this.specPlace = this.$root.$mp.query.groupName
    }
    getLocation().then(res=>{
    console.log(res)
    this.lat = res.latitude;
    this.lng=res.longitude;
    })
    this.getUser();
  },

  components: {
  },

  methods: {
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "添加好友"
      });
    },
    async getUser(){
      const res = await post('User/GetAddfriendInfo',{
          FriendId:this.FriendId,
          UserId:this.userId,
          Token:this.token
      })
      this.userInfo = res.data
    console.log(this.userInfo,'userinfo')
    },
    initData(){
      this.verMsg ='';
    },
    // 添加好友
    async addFriend(){
      const res = await post('User/Addfriend_req',{
        UserId:this.userId,
        Token:this.token,
        FriendId:this.FriendId,
        AuthInfo:this.verMsg,
        Lat:this.lat,
        Lng:this.lng
      })
      if(res.code==0){
        wx.showToast({
          title:'发送成功!等待好友验证'
        })
        setTimeout(()=>{
          wx.navigateBack()
        },1500)
      }
    },
    //选择分组
    choseTeam(){
      wx.navigateTo({
        url:"/pages/connectLetter/addTeam/main?url=messages/addFre"
      })
    },
  },

  created () {
    // let app = getApp()
  }
}
</script>
