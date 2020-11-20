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
    </div>

    <div>
      <ul v-for="(item ,index) in messages" v-bind:key="index +'itemMessage'">
        <li>{{item.user}} says {{item.message}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
  // @ is an alias to /src
  import HelloWorld from "@/components/HelloWorld.vue";
  import * as signalR from "@aspnet/signalr";
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
      };
    },
    methods: {
      // 发送给指定User
      sendOne: function() {
        this.connection
          .invoke("Echo", this.userId, this.message) // invoke 连接后台方法
          .catch(function(err) {
            return console.error(err);
          });
      },
      //给全部发送消息
      sendAll: function() {
        this.connection
          .invoke("SendMessage", this.user, this.message) // invoke 连接后台方法
          .catch(function(err) {
            return console.error(err);
          });
      },
      //只给自己发送消息
      sendOwn: function() {
        this.connection
          .invoke("SendMessageCaller", this.message)
          .catch(function(err) {
            return console.error(err);
          });
      },
      // 加入组
      joinGroup: function() {
        this.connection
          .invoke("JoinGroup", this.groupName)
          .catch(function(err) {
            return console.error(err);
          });
      },
      // 发送信息给所在组
      sendToGroup: function() {
        this.connection
          .invoke("SendToGroup", this.groupName, this.message)
          .catch(function(err) {
            return console.error(err);
          });
      },
      // 离开组
      levelGroup: function() {
        this.connection
          .invoke("LeaveGroup", this.groupName)
          .catch(function(err) {
            return console.error(err);
          });
      },
    },
    created: function() {
      let thisVue = this;
      this.connection = new signalR.HubConnectionBuilder()
        // .withUrl("https://localhost:8789/api/chatHub?username=xxx", { // 带参
        .withUrl("https://localhost:5001/chathub", {
          skipNegotiation: true,
          transport: signalR.HttpTransportType.WebSockets,
        })
        .configureLogging(signalR.LogLevel.Information)
        .build();
      this.connection.on("ReceiveMessage", function(user, message) {  // ReceiveMessage 后台约定的接收方法
        thisVue.messages.push({ user, message });
        console.log({ user, message });
      });
      this.connection.on("ReceiveCaller", function(message) {
        let user = "自己";//这里为了push不报错，我就弄了一个默认值。
        thisVue.messages.push({ user, message });
        console.log({ user, message });
      });
      this.connection.start();
    }
  };
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
