<template>
  <div
    class="page borderTop charRoom"
    id="charRoom"
    :class="showModule==='emotion'?'showEmotion':showModule==='message'?'showMessage':showModule==='imgage'?'showBtn':''"
  >
    <!-- :class="{'showMessage':showModule,'showBtn':showBtn,'showEmotion':showEmotion}" -->
    <!--聊天列表-->
    <div class="padwid" @click="isShowMask=false">
      <div v-for="(msg,msgIndex) in chatList" :key="msgIndex" :id="msg.class">
        <div class="time">{{msg.AddTime}}</div>
        <div class="flex flexAlignCenter boxSize p2 justifyContentEnd plr20" v-if="msg.MsgId=='a'">
          <div class="flex flexAlignEnd justifyContentEnd mrr2">
            <!-- <span class="fontColor" @click="scrollBottom">已读</span> -->
            <!-- <div class="tagmsg cfff"> -->
            <div class="tagmsg" v-if="msg.Info" 
                @longpress="gotouchstart(msg.text)">
              <!-- <text v-if="msg.Info" class="boxSize">{{msg.Info}}</text> -->
              <text class="boxSize" v-html="msg.Info"></text>

              <!-- :onload="scrollBottom()" -->
              <span class="sj rsj"></span>
            </div>
            <img
              class="sendImg"
              mode="widthFix"
              v-if="msg.Pic"
              :src="msg.Pic"
              alt
              @click="previewImg(msg.Pic)"
            />
          </div>
          <div class="avatarbox mr0" v-if="chatStatu.a">
            <img :src="chatStatu.a.Headimgurl" alt class="avatar" @click="goUserCenter" />
          </div>
        </div>
        <div class="flex flexAlignCenter boxSize plr20 justifyContentStart" v-if="msg.MsgId=='b'">
          <div class="avatarbox mr0" v-if="chatStatu.b">
            <img
              :src="chatStatu.b.Headimgurl"
              alt
              class="avatar"
              @click="goUserCenter(chatStatu.b.TempId)"
            />
          </div>
          <div class="flex flexAlignEnd mrl2">
            <!-- <span class="fontColor">已读</span> -->
            <div class="tagmsg bg_fff black" v-if="msg.Info" 
                @longpress="gotouchstart(msg.text)">
              <span class="sj lsj"></span>
              <!-- <p v-if="msg.Info" class="boxSize" style="color:#1a1a1a">{{msg.Info}}</p> -->
              <text
                class="boxSize"
                v-html="msg.Info"
              ></text>
              <!-- :onload="scrollBottom()" -->
            </div>
            <img
              class="sendImg"
              mode="widthFix"
              v-if="msg.Pic"
              :src="msg.Pic"
              alt
              @click="previewImg(msg.Pic)"
            />
          </div>
        </div>
      </div>
    </div>
    <!--底部按钮 输入框 下拉按钮-->
    <div class="bottomicon" v-if="!isTakePhoto">
      <!--常用语按钮-->
      <div class="borderTop usedMes">
        <span
          class="bg_fff"
          @click="getMessage(item.Id)"
          v-for="(item,tindex) in messageType"
          :key="tindex"
        >{{item.Name}}</span>
      </div>
      <div class="inputbtn flex flexAlignCenter bg_fff">
        <div
          class="blur flex1"
          v-if="showModule!=='input'"
          @click="onShowModule('input')"
          :class="sendInfoDiv?'directionR':'color888'"
        >{{sendInfoDiv||'想对他说点什么呢？'}}&#x200E;</div>
        <input
          type="text"
          placeholder="想对他说点什么呢？"
          class="flex1"
          v-model="sendInfo"
          confirm-type="send"
          confirm-hold
          v-else
          @blur="inputFocusStatus=false;showModule=''"
          :focus="inputFocusStatus"
          @confirm="sendMessage"
          :cursor-spacing="10"
        />
        <div class="flex flexAlignCenter">
          <img
            src="/static/images/icons/smile.jpg"
            alt
            class="logimg"
            @click="onShowModule('emotion')"
          />
          <img
            src="/static/images/icons/add.jpg"
            alt
            class="logimg"
            @click="onShowModule('imgage')"
            v-if="!sendInfo"
          />
          <img src="/static/images/icons/send.png" alt class="logimg" @click="sendMessage()" v-else />
        </div>
      </div>
      <!--按钮组-->
      <div v-if="showModule==='imgage'">
        <div class="icon_box flex">
          <div class="flex flexAlignCenter flexColumn" @click="chosseImg('camera')">
            <img src="/static/images/icons/photo.jpg" alt class="icon_put" />
            <p class="fontColor">拍照</p>
          </div>
          <div class="flex flexAlignCenter flexColumn" @click="chosseImg('album')">
            <img src="/static/images/icons/albrem.jpg" alt class="icon_put" />
            <p class="fontColor">相册</p>
          </div>
          <!-- <div class="flex flexAlignCenter flexColumn" @click="goLocation">
            <img src="/static/images/icons/location.jpg" alt class="icon_put">
            <p class="fontColor">位置</p>
          </div>-->
        </div>
      </div>
      <!--常用语-->
      <ul class="messagelist" v-if="showModule==='message'">
        <scroll-view scroll-y @scrolltolower="loadMore" class="scroll_height">
          <van-swipe-cell
            :right-width="65"
            class="swipe-cell"
            async-close
            v-for="(item,index) in messageList"
            :key="index"
          >
            <van-cell-group>
              <van-cell class="chat" @click="onClick(item.Name)">
                <li class="index-group-item">{{item.Name}}</li>
              </van-cell>
            </van-cell-group>
            <span
              slot="right"
              class="van-swipe-cell__right"
              @click.stop="Delete(item.GroupId,item.Id,index)"
            >删除</span>
          </van-swipe-cell>

          <van-swipe-cell class="swipe-cell" v-if="messageList.length<1">
            <div class="noData">还没有常用语呢!</div>
          </van-swipe-cell>
        </scroll-view>
        <li style="color:red" @click="isShowMask=true">添加常用语</li>
      </ul>
      <!-- 表情 -->
      <div class="emotion" v-if="showModule==='emotion'">
        <!-- <emotion @emotion="handleEmotionComment" :height="300" ></emotion> -->

        <scroll-view scroll-y class="emotion-pack-item">
          <div class="emotion-box">
            <!-- <div class="emotion-box-line" v-for="(line, i) in list" :key="i"> -->
            <div
              class="emotion-box-line"
              v-for="(line, i) in emotionArr"
              :key="i"
              @click="handEmotion(line)"
            >
              <!-- <div
                v-for="(item, itemI) in line"
                :key="itemI"
                @click.native="clickHandler(item)"
              >
              {{item}}</div>-->
              <div v-html="line.url"></div>
            </div>
          </div>
        </scroll-view>
      </div>
    </div>
    <!--弹层-->
    <div class="mask" v-if="isShowMask" catchtouchmove="true" @click="isShowMask=false"></div>
    <div class="centerMask" v-if="isShowMask">
      <div class="fontBold mestitle">新增常用语</div>
      <textarea name id cols="30" rows="10" fixed placeholder="输入您的常用回复" v-model="useText"></textarea>
      <div class="flex mesbtn borderTop">
        <p @click="saveText">保存</p>
        <p class="fontColor99" @click="sendText">保存并发送</p>
      </div>
    </div>
  </div>
</template>

<script>
import { post, getCurrentPageUrlWithArgs } from "@/utils";
import emotionList from "@/utils/emotionList";
export default {
  data() {
    return {
      userId: "",
      token: "",
      curPage: "",
      FriendId: "", //好友ID
      TempId: "", //临时联系id
      isShowMask: false, //是否展示遮罩层
      showModule: "", //展示图片组；emotion：表情。message:常用语。imgage:照片
      messageType: [], //常用语分类
      messageList: [], //常用语列表
      addId: "", //添加常用语的标识
      useText: "", //新增的常用语
      chatStatu: {}, //聊天信息
      chatList: [], //聊天列表
      sendInfo: "", //发送消息的内容
      sendInfoDiv: "", //div展示的内容
      // 图片
      imgPathArr: [], //临时路径
      isTakePhoto: false, //是否开启拍照,

      emotionList: emotionList.emotionList,
      emotionArr: [],
      socketStatus: false, //socket状态
      inputFocusStatus: false, //input对焦状态
      page: 1,
      pageSize: 20
    };
  },
  onLoad() {
    console.log(this.emotionList, "emotionList");
    this.setBarTitle();
    this.initEmotion();
    this.curPage = getCurrentPageUrlWithArgs();
    this.initData();
    this.sendInfo = "";
    this.page = 1;
    this.messageList = [];
    this.chatStatu = {};
    this.messageType = [];
    this.FriendId = this.$root.$mp.query.FriendId;
    this.TempId = this.$root.$mp.query.TempId;
    this.getMessageType();
  },
  onShow() {
    this.userId = wx.getStorageSync("userId");
    this.token = wx.getStorageSync("token");
    this.getFriendMessage("scrollBottom").then(() => {
      this.connectSocket();
    });
  },
  onUnload() {
    wx.closeSocket({
      success() {
        console.log("关闭socket成功");
      }
    });
  },
  components: {},
  watch: {
    // 输入消息内容改变时
    sendInfo() {
      if (this.sendInfo.length < 15) {
        this.sendInfoDiv = this.sendInfo;
        return false;
      }
      let maxLength = 33;
      for (let i = 1; i < this.sendInfo.length; i += 1) {
        const str = this.sendInfo.slice(-i);
        let strLength = this.strlength(str);
        if (strLength > maxLength) {
          this.sendInfoDiv = str;
          return false;
          console.log(strLength, str);
        } else if (strLength < maxLength) {
          this.sendInfoDiv = this.sendInfo;
          console.log(strLength, str);
        }
      }
    }
  },
  methods: {
    initData() {
      this.isShowMask = false;
      this.isTakePhoto = false;
      this.inputFocusStatus = false;
      this.addId = "";
      this.useText = "";
      this.showModule = "";
    },
    // 返回字符串长度，中文2，英文1
    strlength(str) {
      let len = 0;
      for (let i = 0; i < str.length; i++) {
        const c = str.charCodeAt(i);
        //单字节加1
        if ((c >= 0x0001 && c <= 0x007e) || (0xff60 <= c && c <= 0xff9f)) {
          len++;
        } else {
          len += 2;
        }
      }

      return len;
    },
    setBarTitle() {
      wx.setNavigationBarTitle({
        title: "对话框"
      });
    },
    // 判断是否会员
    async isVip() {
      const res = await post("User/QueryVipInfo", {
        UserId: this.userId,
        Token: this.token
      });
      const data = res.data;
      // 没开通会员
      if (!data.IsVip) {
        wx.showModal({
          title: "开通会员",
          content: "此功能需要开通会员，是否跳转开通会员页面?",
          confirmColor: "#ff952e",
          cancelColor: "#999",
          success(res) {
            if (res.confirm) {
              wx.navigateTo({ url: "/pages/member2/buyFunction/main?type=3" });
            }
          }
        });
      }
    },
    // 打开webSocket链接
    async connectSocket() {
      const that = this;
      const ress = await post(
        "User/GetWebSocketId",
        {
          UserId: this.userId,
          Token: this.token,
          FriendId: this.FriendId * 1,
          TempId: this.TempId * 1
        },
        this.curPage
      );
      wx.connectSocket({
        url: `wss://ccapi.wtvxin.com/WebSocketServer.ashx?Signature=${
          ress.data.Signature
        }`
      });
      //code=99,是错误，也包过正常错误。
      // code=1 发起登录、登录成功、
      // code=2 发起退出、需要登录
      // code=3 聊天
      // code=0 服务反馈处理了。
      wx.onSocketMessage(message => {
        let msg = JSON.parse(message.data);
        console.log("msg", msg);
        if (msg.code !== 1 && msg.code !== 2) {
          this.page = 1;
          this.getFriendMessage();
        } else {
          this.socketStatus = false;
          this.renewConnectSocket();
        }
      });
      wx.onSocketOpen(res => {
        console.log("open", res);
        this.socketStatus = true;
        this.send({
          MsgType: 1,
          TimeStamp: ress.data.TimeStamp,
          SecretKey: ress.data.SecretKey
        });
      });
      wx.onSocketError(error => {
        console.log("socket error:", error);
        this.socketStatus = false;
        this.pageBack("连接已断开，请重试!");
      });
      wx.onSocketClose(close => {
        console.log("close", close);
        this.socketStatus = false;
        // this.pageBack('连接已断开，请重试!');
      });
    },
    // 断开返回上一页
    pageBack(msg) {
      wx.showToast({ title: msg, icon: "none" });
      setTimeout(() => {
        wx.navigateBack();
      }, 1500);
    },
    // socket断线重连
    renewConnectSocket() {
      // if (!this.socketStatus) {
      //   setTimeout(() => {
      //     this.connectSocket();
      //   }, 3000);
      // }
    },
    // 发送socket
    send(data) {
      console.log("send", JSON.stringify(data));
      wx.sendSocketMessage({ data: JSON.stringify(data) });
    },
    //获取好友消息
    getFriendMessage(scrollBottom, scrollPosition) {
      return new Promise((resolve, reject) => {
        const that = this;
        wx.request({
          url: "https://ccapi.wtvxin.com/api/User/Readfriend_new",
          data: {
            UserId: this.userId,
            Token: this.token,
            FriendId: this.FriendId * 1,
            TempId: this.TempId * 1,
            Page: this.page,
            PageSize: this.pageSize
          },
          method: "POST",
          success(_res) {
            let res = _res.data;
            if (res.code === 0) {
              let info = [];
              // 处理发送时间
              let times =
                res.data.info[0] &&
                new Date(res.data.info[0].AddTime.replace("-", "/"));
              res.data.info.reverse(); //数组翻转
              res.data.info.map((item, i) => {
                let date = new Date(item.AddTime.replace("-", "/"));
                item.class = "class" + date.getTime(); //时间戳类，用户下拉聊天记录滑动对应位置
                if (i !== 0) {
                  const year = date.getFullYear() === times.getFullYear();
                  const Month = date.getMonth() === times.getMonth();
                  const dates = date.getDate() === times.getDate();
                  const Hours = date.getHours() === times.getHours();
                  const Minutes = date.getMinutes() - times.getMinutes() < 5;
                  if (year && Month && dates && Hours && Minutes) {
                    item.AddTime = "";
                  } else {
                    times = date;
                  }
                }
                item.text = item.Info;
                // 将匹配结果替换表情图片
                item.Info = item.Info.replace(
                  /\[[\u4E00-\u9FA5]{1,3}\]/gi,
                  words => {
                    let word = words.replace(/\[|\]/gi, "");
                    let index = that.emotionList.indexOf(word);
                    if (index !== -1) {
                      return `<img style="width:25px;height:25px;" src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/${index}.gif" align="middle">`;
                    } else {
                      return words;
                    }
                  }
                );
                // info.unshift(item);
                info.push(item);
              });

              // res.data.info = info;
              if (that.page === 1) {
                that.chatList = info;
              } else {
                that.chatList = info.concat(that.chatList);
              }
              that.chatStatu = res.data;
              if (
                scrollBottom === "scrollBottom" ||
                (res.data.info.length < that.pageSize && that.page === 1)
              ) {
                console.log("滚动了");
                that.scrollBottom();
              }
              if (scrollPosition) {
                setTimeout(() => {
                  wx
                    .createSelectorQuery()
                    .select("#" + scrollPosition)
                    .boundingClientRect(function(rect) {
                      // 使页面滚动到底部
                      wx.pageScrollTo({
                        scrollTop: rect.top,
                        duration: 0
                      });
                    })
                    .exec();
                }, 100);
              }
              resolve();
            } else if (res.code === 1) {
              wx.showModal({
                title: "开通会员",
                content: "此功能需要开通会员，是否跳转开通会员页面?",
                confirmColor: "#ff952e",
                cancelColor: "#999",
                success(res) {
                  if (res.confirm) {
                    wx.navigateTo({
                      url: "/pages/member2/buyFunction/main?type=3"
                    });
                  } else {
                    wx.navigateBack();
                    reject();
                  }
                }
              });
            } else {
              wx.showToast({ title: res.msg, icon: "none" });
              setTimeout(() => {
                wx.navigateBack();
              }, 1500);
              reject();
            }
          }
        });
      });
    },
    // 发送消息
    async sendMessage(data) {
      let sendInfo = "";
      let imgBase = "";
      let lat = 0;
      let lng = 0;
      console.log(this.sendInfo, "发送消息");
      if (data) {
        // 发送图片
        if (data.type === "img") {
          imgBase = data.data;
        }
        // 发送位置
        else if (data.type === "map") {
          lat = data.data.latitude;
          lng = data.data.longitude;
        } else {
          console.log(this.sendInfo, "发送消息2");
          if (!this.sendInfo) {
            return false;
          }
          sendInfo = this.sendInfo;
        }
      }
      // 普通消息
      else {
        console.log(this.sendInfo, "发送消息2");
        if (!this.sendInfo) {
          return false;
        }
        sendInfo = this.sendInfo;
      }
      const that = this;
      wx.request({
        url: "https://ccapi.wtvxin.com/api/User/Sendfriend_new",
        data: {
          UserId: this.userId,
          Token: this.token,
          FriendId: this.FriendId * 1,
          TempId: this.TempId * 1,
          Info: sendInfo,
          Pic: imgBase,
          Lat: lat,
          Lng: lng
        },
        method: "POST",
        success(res_) {
          that.sendInfo = "";
          const res = res_.data;
          const _res = res.data;
          // 发送成功收回输入框
          // that.showModule==='input'&&(that.showModule = "")
          // that.inputFocusStatus = false;
          that.send({
            MsgType: 3,
            Id: _res.Id,
            Info: sendInfo,
            Pic: _res.Pic,
            AddTime: _res.AddTime,
            Lat: lat,
            Lng: lng
          });
        }
      });
    },
    // 展示模块
    onShowModule(val) {
      console.log(this.showModule, val, "val");
      // if(this.chatStatu.info.length>this.pageSize){
      this.scrollBottom();
      // }
      this.showModule === val
        ? (this.showModule = "")
        : (this.showModule = val);
      setTimeout(() => {
        if (val === "input") {
          this.inputFocusStatus = true;
        } else {
          this.inputFocusStatus = false;
        }
      }, 150);
    },
    // **************************常用语************************
    //获取常用语分类
    getMessageType() {
      post("User/GetUser_wordtype").then(res => {
        if (res.code == 0) {
          this.messageType = res.data;
        }
        // console.log(res,"+++++++++++++++++")
      });
    },
    //点击常用语，获取常用语列表
    getMessage(id, update) {
      console.log("获取常用语");
      // this.initData();
      if (!update) {
        if (this.addId == id) {
          this.onShowModule("message");
          return false;
        } else {
          this.showModule = "message";
        }
      }
      this.addId = id;
      post(
        "User/GetUser_word",
        {
          GroupId: id,
          UserId: this.userId,
          Token: this.token
        },
        this.curPage
      ).then(res => {
        if (res.code == 0) {
          this.messageList = res.data;
        }
        // console.log(res,"+++++++++++++++++")
      });
    },
    // 常用语点击
    onClick(name) {
      this.sendInfo = name;
      this.scrollBottom();
      this.sendMessage();
    },
    //保存常用语
    saveText() {
      // console.log(this.addId,"addId")
      let that = this;
      post(
        "User/AddUser_word",
        {
          UserId: that.userId,
          Token: that.token,
          GroupId: that.addId,
          Info: that.useText
        },
        that.curPage
      ).then(res => {
        if (res.code === 0) {
          wx.showToast({
            title: "新增成功!",
            icon: "none",
            duration: 1500,
            success: function() {
              that.isShowMask = false;
              that.getMessage(that.addId, true);
              console.log(that.messageList, "that.messageList");
              // that.messageList.reverse()
            }
          });
        }
      });
    },
    // 保存并发送常用语
    sendText() {
      this.saveText();
      this.sendInfo = this.useText;
      this.sendMessage();
      this.isShowMask = false;
    },
    //删除常用语
    Delete(GroupId, Id, index) {
      let that = this;
      wx.showModal({
        title: "是否确定删除？",
        confirmColor: "#ff952e",
        cancelColor: "#999",
        success(res) {
          if (res.confirm) {
            that.DelMessageItem(GroupId, Id, index);
          } else if (res.cancel) {
          }
        }
      });
    },
    //确定删除常用语
    DelMessageItem(GroupId, Id, index) {
      let that = this;
      post(
        "User/DelUser_word",
        {
          UserId: this.userId,
          Token: this.token,
          GroupId: GroupId,
          Id: Id
        },
        this.curPage
      ).then(res => {
        // console.log(res,"/////////////////////////////")
        if (res.code === 0) {
          wx.showToast({
            title: "删除成功!",
            icon: "none",
            duration: 1500,
            success: function() {
              that.messageList.splice(index, 1);
              that.getMessage(GroupId);
            }
          });
        }
      });
    },
    // **************************常用语End************************
    //**************************拍照，图片************************** */
    // 选择图片
    chosseImg(sourceType) {
      const that = this;
      let num = 8; //上传的图片最大数量
      wx.chooseImage({
        count: num, //最大图片数量=最大数量-临时路径的数量
        sizeType: ["compressed"], //图片尺寸 original--原图；compressed--压缩图
        sourceType: [sourceType], //选择图片的位置 album--相册选择, 'camera--使用相机
        success: res => {
          this.imgPathArr = res.tempFilePaths;
          console.log(res.tempFilePaths, "base");
          that.updateImg();
        }
      });
    },
    //根据临时路径，获取图片base64码。
    async updateImg() {
      const that = this;
      // 根据临时路径数组imgPathArr获取base64图片
      for (let i = 0; i < this.imgPathArr.length; i += 1) {
        const item = this.imgPathArr[i];
        //异步方法
        const res = wx.getFileSystemManager().readFileSync(item, "base64");
        //成功的回调
        const imgBase = "data:image/png;base64," + res.toString();
        await this.sendMessage({ type: "img", data: imgBase });
      }
    },
    // 预览图片
    previewImg(img) {
      wx.previewImage({
        urls: [img]
      });
    },
    //**************************拍照，图片End************************** */
    // 跳转到选择定位
    // goLocation(){
    //   const that = this;
    //   wx.chooseLocation({
    //     success(res){
    //       console.log(res)
    //       that.sendMessage({type:'map',data:res})
    //     },
    //     fail(err){
    //       console.log(err)
    //     }
    //   })
    // },
    // 初始化表情
    initEmotion() {
      const list = this.emotionList;
      let emotionArr = [];
      list.map((item, index) => {
        emotionArr.push({
          name: `[${item}]`,
          url: `<img style="width:25px;height:25px;" src="https://res.wx.qq.com/mpres/htmledition/images/icon/emotion/${index}.gif">`
        });
      });
      this.emotionArr = emotionArr;
    },
    handEmotion(item) {
      console.log(item, "item");
      this.sendInfo += item.name;
    },
    // 回复add表情
    handleEmotionComment(i) {
      console.log(i, "iiii");
      return false;
      console.log(this.showComment, this.comment[this.showComment], "i");
      this.comment[this.showComment].commentContent += i;
    },
    // 滚动到底部
    scrollBottom() {
      setTimeout(() => {
        wx
          .createSelectorQuery()
          .select("#charRoom")
          .boundingClientRect(function(rect) {
            // 使页面滚动到底部
            wx.pageScrollTo({
              scrollTop: rect.height,
              duration: 0
            });
            console.log(rect, "height滚动了额");
          })
          .exec();
      }, 100);
    },
    // 查看个人资料
    goUserCenter(id) {
      console.log("typeOf(id*1)", typeof id, id);
      if (typeof id === "number") {
        wx.navigateTo({ url: "/pages/mine/person/main?type=2&Id=" + id });
      } else {
        wx.navigateTo({ url: "/pages/mine/person/main?type=1" });
      }
    },
    // ******************长按复制***************
    gotouchstart(data) {
        console.log('复制',data)
        wx.setClipboardData({data})
    },
  },
  //下拉刷新
  onPullDownRefresh() {
    this.initData();
    this.page += 1;

    this.getFriendMessage("", this.chatList[0].class);
    wx.stopPullDownRefresh();
  },
  created() {
    // let app = getApp()
  }
};
</script>
<style scoped lang="scss">
.padwid {
  // height:86vh;
  // width:100%;
  // box-sizing:border-box;
  // padding-bottom:180rpx;
  .time {
    color: #888;
    font-size: 20rpx;
    text-align: center;
    margin: 10rpx 0;
  }
}
.showMessage {
  padding-bottom: 530rpx !important;
}
.showBtn {
  padding-bottom: 400rpx !important;
}
.showEmotion {
  padding-bottom: 480rpx !important;
}
.plr20 {
  padding: 20rpx !important;
}
.mrr2 {
  margin-right: 6rpx !important;
  width: 80%;
}
.mrl2 {
  margin-left: 6rpx !important;
  width: 80%;
}
.charRoom {
  padding-bottom: 180rpx;
}
.bottomicon {
  background: #f4f4f4;
}
.usedMes {
  padding: 20rpx 30rpx !important;
  span {
    border-radius: 20rpx;
    padding: 10rpx 22rpx;
    font-size: 26rpx;
    margin-left: 10rpx;
  }
}
.messagelist {
  height: 350rpx;
  .scroll_height {
    height: 250rpx;
  }
  li {
    height: 80rpx;
    line-height: 80rpx;
    text-align: center;
    color: #1a1a1a;
    border-top: 1rpx solid #ececec;
  }
}
.centerMask {
  width: 650rpx;
  height: 400rpx;
  top: 50%;
  left: 50rpx;
  margin-top: -200rpx;
  position: fixed;
  z-index: 9999;
  background: #fff;
  border-radius: 10rpx;
  padding: 0 30rpx;
  box-sizing: border-box;
  textarea {
    margin-top: 20rpx;
    height: 200rpx;
  }
  .mestitle {
    margin-top: 30rpx;
  }
}
.mesbtn {
  p {
    width: 50%;
    padding: 30rpx;
    box-sizing: border-box;
    text-align: center;
  }
  p:last-child {
    border-left: 1rpx solid #ececec;
  }
}
.van-swipe-cell__left,
.van-swipe-cell__right {
  display: inline-block;
  width: 130rpx;
  height: 80rpx;
  font-size: 28rpx;
  line-height: 80rpx;
  color: #fff;
  text-align: center;
  background-color: #f44;
}
.index-group-item {
  padding: 0;
}
.noData {
  color: #999;
}
.sendImg {
  width: 200rpx;
  height: auto;
}
// 拍照
.takePhoto {
  background: #000;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 99999;
  .photo {
    width: 100%;
    height: 90vh;
  }
  .take {
    color: #fff;
    font-size: 40rpx;
    width: 100%;
    height: 10vh;
    line-height: 10vh;
    display: flex;
    align-items: center;
    justify-content: center;
    img {
      border-radius: 50%;
      width: 80rpx;
      height: 80rpx;
    }
  }
}
.tagmsg {
  padding: 15rpx;
  font-size: 30rpx;
  word-break: break-all;
  background: #fff;
  color: #000;
  p {
    word-break: break-all;
  }
}
.rsj {
  background: #fff;
  border: 6rpx solid #fff;
}
// 表情
.emotion {
  height: 300rpx;
  position: relative;
}
.emotion-pack-item {
  height: 300rpx;
}
.emotion-box {
  display: flex;
  align-items: center;
  flex-flow: row wrap;
}
.emotion-box-line {
  margin: 10rpx;
  img {
    width: 40rpx;
    height: 40rpx;
  }
}
.cfff {
  color: #fff;
}
.black {
  color: #333;
}
input {
  position: relative;
  height: 58rpx;
  line-height: 58rpx;
  white-space: wrap;
  width: 480rpx;
}
.blur {
  border: 1rpx solid #f2f2f2;
  padding: 10rpx 20rpx;
  border-radius: 10rpx;
  background: #f4f4f4;
  height: 58rpx;
  line-height: 58rpx;
  text-align: left;
  width: 500rpx;
  overflow: hidden;
  white-space: nowrap;
}
.directionR {
  // direction:rtl;//文字右对齐，隐藏左边
}
.color888 {
  color: #888;
}
</style>
