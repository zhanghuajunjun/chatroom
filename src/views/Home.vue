<template>
  <div class="box">
    <div class="loginBox" :class="{'active':flag===false}">
      <div class="title">用户登录</div>
      <div class="nav">用户名</div>
      <input type="text" placeholder="请输入用户名" v-model="username" class="input" />
      <div class="nav">选择头像</div>
      <div class="avatar">
        <div v-for="item in imgs" :key="item.id" class="item" @click="clickImg(item.id,item.url)">
          <img :src="item.url" alt class="img" :class="{'now': id == item.id}" />
        </div>
      </div>
      <button @click="login" class="button">登录</button>
    </div>
    <div class="chatbox" :class="{'actives':flag===false}">
      <div class="chat">
        <div class="left">
          <div class="userinfo">
            <img :src="avatar" class="avatar_img" />
            <div class="username">{{name}}</div>
          </div>
          <div class="userlist">用户列表</div>
          <div>
            <div class="info" v-for="(item,index) in userList" :key="index">
              <img :src="item.avatar" class="avatar_img" />
              <div class="username">{{item.username}}</div>
            </div>
          </div>
        </div>
        <div class="right">
          <div class="chattitle">聊天室（{{num}}）</div>
          <div class="content" id="contentBox">
            <div v-for="(item,index) in MSG" :key="index">
              <div class="jion" v-if="item.notice">{{item.notice}}</div>
              <div class="jion" v-else-if="item.leave">{{item.leave}}</div>
              <div v-else>
                <div class="else" v-if="item.username !== name">
                  <img :src="item.avatar" class="avatar_img" />
                  <div>
                    <div class="other_name">{{item.username}}</div>
                    <div v-if="item.img">
                      <img :src="item.img" alt />
                    </div>
                    <div v-else>
                      <div class="else_container" v-html="item.content"></div>
                    </div>
                  </div>
                </div>
                <div class="self" v-else>
                  <div v-if="item.img" class="ssssss">
                    <img :src="item.img" alt class="photo_img" />
                  </div>
                  <div v-else>
                    <div class="self_container" v-html="item.content"></div>
                  </div>
                  <img :src="item.avatar" class="avatar_img" />
                </div>
              </div>
            </div>
            <div class="emojione" v-if="checked == true">
              <picker set="emojione" @select="addEmoji" />
            </div>
          </div>
          <div class="chat_footer">
            <div class="imgs">
              <img src="../image/表情.png" alt class="eimg" @click="pickEmoji" />
              <img src="../image/042剪刀-1.png" alt class="eimg" />
              <div class="photo">
                <img src="../image/图片.png" alt class="eimg" />
                <input type="file" @change="onChange" class="choose_img" />
              </div>
            </div>
            <div
              ref="content"
              class="ipt"
              contenteditable
              @keydown.enter.prevent="sendMessage"
              @focus="onFocus"
            ></div>
            <div class="sendmsg">
              <div class="send">按下Enter发送</div>
              <button @click="sendMessage">发送</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { Picker } from "emoji-mart-vue";
export default {
  name: "",
  props: {},
  data() {
    return {
      content: "",
      num: 0,
      id: "",
      username: "",
      url: "",
      imgs: [
        { id: "1", url: require("../image/timgs.jpg") },
        { id: "2", url: require("../image/timg.jpg") },
        { id: "3", url: require("../../server/public/images/avatar03.jpg") },
        { id: "4", url: require("../../server/public/images/avatar04.jpg") },
        { id: "5", url: require("../../server/public/images/avatar05.jpg") },
        { id: "6", url: require("../../server/public/images/avatar06.jpg") },
        { id: "7", url: require("../../server/public/images/avatar07.jpg") },
        { id: "8", url: require("../../server/public/images/avatar08.jpg") },
        { id: "9", url: require("../../server/public/images/avatar09.jpg") },
        { id: "10", url: require("../../server/public/images/avatar10.jpg") },
      ],
      userList: [],
      name: "",
      avatar: "",
      flag: true,
      MSG: [],
      checked: false,
      pickUrl: "",
    };
  },
  components: {
    Picker,
  },
  methods: {
    clickImg(id, url) {
      this.id = id;
      this.url = url;
    },
    login() {
      if (this.username === "" || this.url === "") {
        alert("用户名或头像不能为空");
      } else {
        this.$socket.emit("login", {
          username: this.username,
          avatar: this.url,
        });
      }
    },
    sendMessage() {
      if (this.pickUrl !== "" && this.$refs.content.innerText === "") {
        this.$socket.emit("sendImage", {
          img: this.pickUrl,
          username: this.username,
          avatar: this.url,
        });
      } else {
        this.$socket.emit("sendMessage", {
          content: this.$refs.content.innerHTML,
          username: this.username,
          avatar: this.url,
        });
      }
      this.$refs.content.innerHTML = "";
      this.pickUrl = "";
    },
    pickEmoji() {
      this.checked = true;
    },
    onFocus() {
      this.checked = false;
    },
    addEmoji(e) {
      this.$refs.content.innerHTML += e.native;
    },
    onChange(e) {
      let file = e.target.files[0];
      let fr = new FileReader();
      fr.readAsDataURL(file);
      fr.onload = () => {
        this.pickUrl = fr.result;
        this.$refs.content.innerHTML += `<img src=${fr.result} width="60%" hight="60%">`;
      };
    },
  },
  sockets: {
    loginSuccess(data) {
      this.name = data.username;
      this.avatar = data.avatar;
      this.flag = false;
    },
    loginError(data) {
      alert("用户名重复，登陆失败");
    },
    userList(data) {
      this.num = data.length;
      if (data.length > this.userList.length) {
        let obj = {};
        obj.notice = `“${data[data.length - 1].username}”加入了群聊`;
        this.MSG.push(obj);
      }
      this.userList = data;
    },
    receiveMessage(data) {
      this.MSG.push(data);
    },
    receiveImage(data) {
      this.MSG.push(data);
    },
    delUser(data) {
      let obj = {};
      obj.leave = `“${data.username}”离开了群聊`;
      this.MSG.push(obj);
    },
  },
  mounted() {},
  watch: {
    MSG() {
      let obj = document.getElementById("contentBox");
      this.$nextTick(() => {
        obj.scrollTop = obj.scrollHeight;
      });
    },
  },
  computed: {},
};
</script>

<style scoped lang='scss'>
.box {
  display: flex;
  justify-content: center;
}
.loginBox {
  margin-top: 100px;
  width: 370px;
  padding: 30px;
  background: #eeeeee;
}
.title {
  font-size: 24px;
  font-weight: 600px;
  margin-bottom: 30px;
  text-align: center;
}
.nav {
  font-size: 18px;
  margin: 5px 0;
}
.input {
  width: 362px;
  height: 30px;
}
.avatar {
  width: 350px;
  display: flex;
  flex-wrap: wrap;
  padding-right: 18px;
  border: 1px solid #000;
  padding-bottom: 20px;
}
.item {
  margin-left: 20px;
  width: 50px;
  height: 50px;
  margin-top: 20px;
}
.img {
  width: 48px;
  height: 48px;
}
.ssssss {
  display: flex;
  justify-content: flex-end;
  margin-right: 10px;
}
.button {
  width: 370px;
  height: 40px;
  background: skyblue;
  font-size: 20px;
  margin-top: 10px;
  border-color: skyblue;
}
.now {
  border: 1px solid green;
}
.chatbox {
  display: none;
}
.active {
  display: none;
}
.actives {
  display: block;
}
.chat {
  display: flex;
}
.avatar_img {
  width: 40px;
  height: 40px;
}
.left {
  padding: 20px;
  width: 220px;
  height: 640px;
  background: rgba($color: #000000, $alpha: 0.5);
}
.userinfo {
  display: flex;
  align-items: center;
}
.username {
  color: white;
  margin-left: 20px;
}
.userlist {
  color: white;
  font-size: 20px;
  padding-top: 20px;
}
.info {
  display: flex;
  align-items: center;
  margin-top: 20px;
}
.right {
  width: 630px;
  height: 660px;
  padding: 10px 0;
  background: #eeeeee;
  position: relative;
}
.chattitle {
  text-align: center;
  font-size: 20px;
  padding-bottom: 10px;
  border-bottom: 1px solid #999999;
}
.emojione {
  position: absolute;
  bottom: 0;
  left: 0;
}
.bbbbbbbbbb {
  display: flex;
  justify-content: flex-end;
}
.else_container {
  word-break: break-all;
  font-size: 14px;
  background: greenyellow;
  padding: 8px 10px;
  margin-left: 10px;
  border-radius: 5px;
}
.else {
  display: flex;
  justify-content: flex-start;
  margin-top: 5px;
}
.other_name {
  margin-left: 10px;
  margin-bottom: 5px;
  font-size: 12px;
  color: #333;
}
.self {
  display: flex;
  justify-content: flex-end;
  margin-top: 5px;
}
.self_container {
  word-break: break-all;
  font-size: 14px;
  background: skyblue;
  padding: 8px 10px;
  margin-right: 10px;
  border-radius: 5px;
}
.content {
  position: relative;
  padding: 10px;
  overflow: auto;
  height: 447px;
}
.jion {
  font-size: 14px;
  text-align: center;
  padding-bottom: 10px;
  color: #999;
}
.chat_footer {
  border-top: 1px solid #999;
  padding: 10px 20px;
}
.imgs {
  display: flex;
  margin-bottom: 10px;
}
.photo_img {
  width: 60%;
  border-radius: 5px;
}
.eimg {
  width: 20px;
  height: 20px;
  margin-right: 20px;
}
.ipt {
  width: 590px;
  height: 80px;
  outline: none;
  resize: none;
  border: none;
  overflow: auto;
  background-color: #eeeeee;
}
.send {
  color: #999;
  background: #eeeeee;
  font-size: 14px;
  margin-right: 10px;
}
.sendmsg {
  margin-top: 10px;
  display: flex;
  justify-content: flex-end;
}
.photo {
  position: relative;
  width: 50px;
}
.choose_img {
  // （opacity）透明度 0代表完全透明 1代表不透明
  opacity: 0;
  z-index: 99;
  position: absolute;
  left: 0;
}
</style>
