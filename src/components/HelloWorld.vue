<script setup>
import { ref,onMounted } from 'vue'
import SockJS from 'sockjs-client/dist/sockjs'
import Stomp from "stompjs";

let isConnected=ref(false);
const socketEndpoint = ref("http://localhost:8080/ws")
let stompClient
let receiver=ref('')
let sender=ref('1')
let queneMessage=ref('')
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
  stompClient.send("/app/sendToUser/" + receiver.value, {name: '1'}, queneMessage.value);
  messages.value.push({
    sender: sender.value,
    content: queneMessage.value,
  });
}
function connectWebSocket() {
  let socket = new SockJS(socketEndpoint.value);

  stompClient = Stomp.over(socket);

  stompClient.connect(
    {name: '1'},
    () => {
      isConnected.value = true;
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

      stompClient.subscribe("/gogo/greeting", (res) => {
        const messageData = JSON.parse(res.body);
        messages.value.push({
          sender: "Server",
          content: messageData,
        });
      });
    },
    (error) => {
      console.log("连接失败: " + error);
      isConnected.value = false; // 更新连接状态
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
      isConnected.value = false; // 更新连接状态
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
        <el-card style="width:400px">
          <el-row>
            <el-col :span="14">
              <el-input v-model="socketEndpoint" class="w-100px" />
              <el-button
                type="primary"
                class="ml-5"
                @click="connectWebSocket"
                :disabled="isConnected"
                >连接</el-button
              >
              <el-button
                type="danger"
                @click="disconnectWebSocket"
                :disabled="!isConnected"
                >断开</el-button
              >
            </el-col>

            <el-col :span="8" class="text-right">
              连接状态：
              <el-tag class="ml-2" type="success" v-if="isConnected"
                >已连接</el-tag
              >
              <el-tag class="ml-2" type="info" v-else>已断开</el-tag>
            </el-col>
            
          </el-row>
        </el-card>
    <el-card class="mt-5">
          <el-form label-width="90px">
            <el-form-item label="消息内容">
              <el-input type="textarea" v-model="topicMessage" />
            </el-form-item>

            <el-form-item>
              <el-button @click="sendToAll" type="primary">发送广播</el-button>
            </el-form-item>
          </el-form>
        </el-card>
        <el-card class="mt-5">
          <el-form label-width="90px">
            <el-form-item label="消息内容">
              <el-input type="textarea" v-model="queneMessage" />
            </el-form-item>
            <el-form-item label="消息接收人">
              <el-input v-model="receiver" />
            </el-form-item>
            <el-form-item>
              <el-button @click="sendToUser" type="primary"
                >发送点对点消息</el-button
              >
            </el-form-item>
          </el-form>
        </el-card>
          
        
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.message-container {
  display: flex;
  flex-direction: column;
}

.message {
  padding: 10px;
  margin: 10px;
  border-radius: 5px;
}

.message--sent {
  align-self: flex-end;
  background-color: #dcf8c6;
}

.message--received {
  align-self: flex-start;
  background-color: #e8e8e8;
}

.message-content {
  display: flex;
  flex-direction: column;
}

.message-sender {
  margin-bottom: 5px;
  font-weight: bold;
  text-align: right;
}

.message-receiver {
  margin-bottom: 5px;
  font-weight: bold;
  text-align: left;
}

.tip-message {
  align-self: center;
  padding: 5px 10px;
  margin-bottom: 5px;
  font-style: italic;
  text-align: center;
  background-color: #f0f0f0;
  border-radius: 5px;
}
</style>
