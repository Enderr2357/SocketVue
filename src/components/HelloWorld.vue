<script setup>
import { ref,onMounted } from 'vue'
import SockJS from 'sockjs-client/dist/sockjs'
import Stomp from "stompjs";

let isConnected=false;
const socketEndpoint = ref("http://localhost:8080/ws")
let stompClient

const MessageType={
  type:String,
  sender:String,
  content:String
}
// const messages = ref<MessageType>([]);
  const messages = ref([]);
const topicMessage = ref(
  "亲爱的大冤种们，由于一只史诗级的BUG，系统版本已经被迫回退到了0.0.1。"
); 
function sendToAll() {
  stompClient.send("/app/sendToAll", {}, topicMessage.value);
  messages.value.push({
    sender: "123123",
    content: topicMessage.value,
  });
}

function sendToUser() {
  stompClient.send("/app/sendToUser/" + receiver.value, {}, queneMessage.value);
  messages.value.push({
    sender: receiver.value,
    content: queneMessage.value,
  });
}
function connectWebSocket() {
  let socket = new SockJS(socketEndpoint.value);

  stompClient = Stomp.over(socket);

  stompClient.connect(
    {Authorization:'bearer'},
    () => {
      isConnected = true;
      messages.value.push({
        sender: "Server",
        content: "Websocket 已连接",
        type: "tip",
      });

      stompClient.subscribe("/topic/notice", (res) => {
        messages.value.push({
          sender: "Server",
          content: res.body,
        });
      });

      stompClient.subscribe("/user/queue/greeting", (res) => {
        const messageData = JSON.parse(res.body);
        messages.value.push({
          sender: messageData.sender,
          content: messageData.content,
        });
      });
    },
    (error) => {
      console.log("连接失败: " + error);
      isConnected = false; // 更新连接状态
      messages.value.push({
        sender: "Server",
        content: "Websocket 已断开",
        type: "tip",
      });
    }
  );
}

function disconnectWebSocket() {
  if (stompClient && stompClient.connected) {
    stompClient.disconnect(() => {
      isConnected = false; // 更新连接状态
      messages.value.push({
        sender: "Server",
        content: "Websocket 已断开",
        type: "tip",
      });
    });
  }
}

onMounted(() => {
  connectWebSocket();
});

const count = ref(0)
</script>

<template>
  <el-button type="primary" @click="connectWebSocket" :disabled="isConnected">连接</el-button>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
