<template>
  <div class="home">
    <h1>前端演示SignalR</h1>
    <div class="content">
      <label class="tip">ToUser</label>
      <input v-model="user" type="text" />
      <label class="tip">ToUserId</label>
      <input v-model="userId" type="text" />
    </div>
    <div class="content" style="margin: 10px auto">
      <label class="tip">Message</label>
      <input v-model="message" type="text" />
      <button @click="sendAll">发送全部</button>
      <button @click="sendOwn">发给自己</button>
      <button @click="sendToGroup">发送组</button>
      <button @click="sendOne">发给指定Id</button>
    </div>
    <div class="content">
      <label class="tip">Group</label>
      <input v-model="groupName" type="text" />
      <button @click="joinGroup">加入组</button>
      <button @click="levelGroup">离开组</button>
      <button @click="close">断开连接</button>
    </div>
    <div>
      <ul v-for="(item ,index) in messages" v-bind:key="index +'itemMessage'">
        <li>{{item.user}} says {{item.message}}</li>
      </ul>
    </div>

    <hr>
    <div>
      <h3>登录后使用SignalR</h3>
      <div class="content">
        <label class="tip">UserName</label>
        <input v-model="loginName" type="text" />
      </div>
      <div class="content" style="margin: 10px auto">
        <label class="tip">Password</label>
        <input v-model="password" type="text" @keyup.13="login"/>
      </div>
      <div class="content" >
        <span style="display: inline-block; width: 80px"></span>
        <button style="padding: 0 30px" @click="getTestManagement" >getTestManagement</button>
      </div>
    </div>
  </div>
</template>

<script>
  // @ is an alias to /src
  import HelloWorld from "@/components/HelloWorld.vue";
  import * as signalR from "@aspnet/signalr";
  import axios from 'axios';

  export default {
    name: "Home",
    components: {
      HelloWorld
    },
    data() {
      return {
        user: "", //用户
        userId: "", //用户Id
        message: "", //消息
        connection: "", //signalr连接
        messages: [], //返回消息
        groupName: "", // 组名
        loginName: '',
        password: '',
      };
    },
    methods: {
      getTestManagement: function(){
        axios.get('https://localhost:8789/api/v0.1/Test/TestHub')
          .then(function (response) {
            console.log(response);
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      // 登录
      login: function() {
        let that = this;
        axios.post('https://localhost:8789/api/v0.1/LoginManagement/LogInWithAccount', {
          LoginName: this.loginName,
          Password: this.password,
        })
          .then(function (response) {
            that.connect();
            console.log(response);
          })
          .catch(function (error) {
            console.log(error);
          });
      },
      // 发送给指定User
      sendOne() {
        this.connection
          .invoke("Echo", this.userId, this.message) // invoke 连接后台方法
          .catch(function (err) {
            return console.error(err);
          });
      },
      //给全部发送消息
      sendAll() {
        this.connection
          .invoke("SendMessage", this.user, this.message) // invoke 连接后台方法
          .catch(function (err) {
            return console.error(err);
          });
      },
      //只给自己发送消息
      sendOwn() {
        this.connection
          .invoke("SendMessageCaller", this.message)
          .catch(function (err) {
            return console.error(err);
          });
      },
      // 加入组
      joinGroup() {
        this.connection
          .invoke("JoinGroup", this.groupName)
          .catch(function (err) {
            return console.error(err);
          });
      },
      // 发送信息给所在组
      sendToGroup() {
        this.connection
          .invoke("SendToGroup", this.groupName, this.message)
          .catch(function (err) {
            return console.error(err);
          });
      },
      close() {
        console.log('I am close');
        this.connection
          .invoke("Close")
          .catch(function (err) {
            return console.error(err);
          });
      },
      // 离开组
      levelGroup() {
        this.connection
          .invoke("LeaveGroup", this.groupName)
          .catch(function (err) {
            return console.error(err);
          });
      },
      // created: function() {  // 进来直接连接
      connect: function() {
        console.log('Hello 我已经开始连接了');
        let thisVue = this;
        this.connection = new signalR.HubConnectionBuilder()
          // .withUrl("https://localhost:8789/api/chatHub?username=xxx", { // 带参, 注意路径不能在普通路径下（api/v0.1/）
          .withUrl("https://localhost:8789/api/chatHub?LoginName=" + this.loginName, {
            skipNegotiation: true,
            transport: signalR.HttpTransportType.WebSockets,
          })
          .configureLogging(signalR.LogLevel.Information)
          .build();
        this.connection.on("ReceiveMessage", function (user, message) {  // ReceiveMessage 后台约定的接收方法
          thisVue.messages.push({user, message});
          console.log({user, message});
        });
        this.connection.on("ReceiveCaller", function (message) {
          let user = "自己";//这里为了push不报错，我就弄了一个默认值。
          thisVue.messages.push({user, message});
          console.log({user, message});
        });
        // this.connection.start({ pingInterval: 300000 }); // 心跳时间，单位毫秒，默认五分钟
        this.connection.start(); // 默认不操作，自动断开时间，单位毫秒
        this.connection.onclose((error) => {
          console.log('连接已关闭');
        });
      },
    }
  }
</script>
<style>
  .content{
    width: 560px;
    text-align: left;
    /*background-color: #bfa;*/
    margin: 0 auto;
  }
  .tip{
    display: inline-block;
    width: 80px;
  }

</style>
