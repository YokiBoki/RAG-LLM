<template>
  <div class="ai-practice-container">
    <!-- 左侧历史对话记录 -->
    <div class="history-panel">
      <div class="new-chat-container">
        <button class="new-chat-btn" @click="newConversation">
          新建对话
          <el-icon class="plus-icon"><Plus /></el-icon>
        </button>
      </div>
      <ul class="history-list">
        <li v-for="(item, index) in historyList" :key="index" @click="selectConversation(index)" :class="{ active: currentConversationIndex === index }">
          {{ item.title }}
        </li>
      </ul>
    </div>

    <!-- 右侧对话页面 -->
    <div class="chat-wrapper">
      <div class="chat-panel">
        <!-- 上半部分聊天界面 -->
        <div class="chat-messages" ref="chatMessagesRef">
          <div v-for="(message, index) in currentConversation.messages" :key="index" :class="['message', message.role]">
            <div class="avatar">
              <div v-if="message.role !== 'user'" class="ai-avatar">
                <img src="@/assets/images/hailuo2.png" alt="AI Avatar">
              </div>
              <div v-else>
                <img src="@/assets/images/user.png" alt="Me">
              </div>
            </div>
            <div class="content">
              {{ message.content }}
<!--              <audio v-if="message.audioUrl" :src="message.audioUrl" controls></audio>-->
              <!-- <AudioBase></AudioBase> -->
            </div>
          </div>
        </div>

        <!-- 输入框 -->
        <div class="input-area">
          <div class="input-wrapper">
            <el-icon class="input-icon link-icon"><Link /></el-icon>
            <input
              v-model="userInput"
              @keyup.enter="sendMessage"
              placeholder="输入消息，按回车发送..."
              type="text"
              :disabled="isInputDisabled"
            >
            <div class="button-group">
              <div class="audio-wave" v-if="isRecording" @click="finishRecording">
                <span v-for="n in 4" :key="n" :style="{ animationDelay: `${n * 0.2}s` }"></span>
              </div>
              <el-icon v-else class="input-icon microphone-icon" @click="toggleRecording">
                <Microphone />
              </el-icon>
              <div class="separator"></div>
              <el-popover
                placement="top"
                :width="200"
                trigger="hover"
                :disabled="!!userInput.trim()"
              >
                <template #reference>
                  <el-button 
                    class="send-button" 
                    circle 
                    @click="sendMessage"
                    :disabled="!userInput.trim()"
                  >
                    <el-icon><Top /></el-icon>
                  </el-button>
                </template>
                <span>请文字/录音/上传语音回复</span>
              </el-popover>
            </div>
          </div>
        </div>
        
        <div class="disclaimer">服务生成的所有内容均由魔法海螺生成，其生成内容的准确性和完整性无法保证，不代表UP主的态度或观点</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick, onMounted, onUnmounted } from 'vue';
import { Link, Microphone } from '@element-plus/icons-vue';
// import AudioBase from "@/components/AudioBase.vue";

const historyList = ref([
  { 
    title: '蟹堡制作咨询',
    messages: [
      { role: 'assistant', content: '您好！我是蟹堡王的神奇海螺，我注意到您最近在制作蟹堡时遇到了一些困难？' },
      { role: 'user', content: '是的，我的蟹堡总是做不好，口感跟章鱼哥做的差太多了。' },
      { role: 'assistant', content: '别担心！让我来帮您分析一下。首先，您是否严格按照蟹堡秘方的比例在调配调味料呢？另外，确保使用新鲜的海藻面包也很重要。' }
    ]
  },
  // { 
  //   title: '比奇堡特色菜单咨询',
  //   messages: [
  //     { role: 'user', content: '请问比奇堡最受欢迎的菜品有哪些？' },
  //     { role: 'assistant', content: '当然是蟹堡王的招牌美食啦！我们的经典蟹堡、海藻薯条和珊瑚果汁都非常受欢迎。特别是蟹堡，采用秘制酱料配方，每天都有无数顾客专程来品尝呢！' }
  //   ]
  // },
  // { 
  //   title: '配送服务咨询',
  //   messages: [
  //     { role: 'user', content: '你们有送餐服务吗？我在水母荡。' },
  //     { role: 'assistant', content: '是的，我们提供送餐服务！水母荡在我们的配送范围内。派大星是我们最可靠的送餐员，保证30分钟内送达，否则您可以获得一个免费的海藻汉堡！' }
  //   ]
  // },
  { 
    title: '这里都是前端写死的',
  },
  { 
    title: '因为后端还没接数据库',
    },
    { 
      title: '大家要用的话自己改一下',
    }
]);

const currentConversationIndex = ref(0);
const userInput = ref('');
const isListening = ref(false);
const chatMessagesRef = ref(null);
const isRecording = ref(false);
let mediaRecorder = null;
let audioChunks = [];
const mediaStream = ref(null);
const isInputDisabled = ref(false);

const currentConversation = computed(() => historyList.value[currentConversationIndex.value]);

const selectConversation = (index) => {
  currentConversationIndex.value = index;
  nextTick(() => {
    scrollToBottom();
  });
};

const newConversation = () => {
  historyList.value.unshift({
    title: '新对话',
    messages: [{ role: 'assistant', content: '您好！我是蟹堡王的神奇海螺，很高兴为您服务！我可以回答关于蟹堡王和汉堡制作的任何问题，您有什么需要帮助的吗？' }]
  });
  currentConversationIndex.value = 0;
};

const sendMessage = () => {
  if (userInput.value.trim()) {
    currentConversation.value.messages.push({ role: 'user', content: userInput.value });
    userInput.value = '';
    nextTick(() => {
      scrollToBottom();
    });
    // 模拟AI回复
    setTimeout(() => {
      currentConversation.value.messages.push({ 
        role: 'assistant', 
        content: '作为蟹堡王的神奇海螺，我很乐意为您解答关于美味蟹堡的任何问题。不过请记住，蟹堡的秘方配方是绝密哦！' 
      });
      nextTick(() => {
        scrollToBottom();
      });
    }, 1000);
  }
};

const finishRecording = () => {
  if (isRecording.value && mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop();
    isRecording.value = false;
    isInputDisabled.value = false;
  }
};

const sendAudioMessage = (audioBlob) => {
  const audioUrl = URL.createObjectURL(audioBlob);
  currentConversation.value.messages.push({
    role: 'user',
    content: '发送了一条语音消息',
    audioUrl: audioUrl
  });
  nextTick(() => {
    scrollToBottom();
  });
  
};

const toggleRecording = async () => {
  // if (!isRecording.value) {
  //   try {
  //     mediaStream.value = await navigator.mediaDevices.getUserMedia({ audio: true });
  //     mediaRecorder = new MediaRecorder(mediaStream.value);
  //     audioChunks = [];

  //     mediaRecorder.ondataavailable = (event) => {
  //       audioChunks.push(event.data);
  //     };

  //     mediaRecorder.onstop = () => {
  //       const audioBlob = new Blob(audioChunks, { type: 'audio/wav' });
  //       console.log('录音已保存', audioBlob);
  //       sendAudioMessage(audioBlob);
  //       stopMediaStream();
  //       isInputDisabled.value = false;
  //     };

  //     mediaRecorder.start();
  //     isRecording.value = true;
  //     isInputDisabled.value = true;
  //   } catch (err) {
  //     console.error('无法访问麦克风', err);
  //   }
  // } else {
  //   finishRecording();
  // }
};

const stopMediaStream = () => {
  if (mediaStream.value) {
    mediaStream.value.getTracks().forEach(track => track.stop());
    mediaStream.value = null;
  }
};

const scrollToBottom = () => {
  const chatMessages = chatMessagesRef.value;
  chatMessages.scrollTop = chatMessages.scrollHeight;
};

const visitorName = ref('Test Customer');

onMounted(() => {
});

onUnmounted(() => {
  finishRecording();
  stopMediaStream();
});
</script>

<style scoped>
/* 样式保持不变 */
.ai-practice-container {
  display: flex;
  height: 100vh;
  font-family: Arial, sans-serif;
}

.history-panel {
  width: 280px;
  background: linear-gradient(135deg, rgba(230, 240, 255, 0.01), rgba(240, 230, 255, 0.01));
  background-color: #ffffff;
  padding: 20px;
  overflow-y: auto;
}

.new-chat-container {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.new-chat-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 10px; /* 略微增加内边距 */
  margin-top: 5px;
  margin-bottom: 5px;
  background: linear-gradient(to right, #0069e0, #0052bc); /* 改用更深的蓝色渐变 */
  color: white;
  border: none;
  border-radius:8px;
  cursor: pointer;
  transition: opacity 0.3s;
  font-size: 14px; /* 加大字号 */
  font-weight: bold; /* 加粗字体 */
}

.new-chat-btn:hover {
  opacity: 0.9;
}

.history-list {
  list-style-type: none;
  padding: 0;
}

.history-list li {
  padding: 10px;
  margin-bottom: 10px;
  background-color: #ffffff;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.history-list li:hover,
.history-list li.active {
  background-color: rgba(0, 105, 224, 0.15);
  color: #0052bc;
}

.chat-wrapper {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, 
    rgba(0, 105, 224, 0.08),
    rgba(0, 56, 148, 0.08)
  );
}

.chat-panel {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background-color: transparent;
  box-shadow: none;
  padding-top: 20px; /* 添加顶部内边距 */
  /* padding-left: 10%;
  padding-right: 10%; */
}

.visitor-info {
  background-color: transparent; /* 背透明 */
  padding: 15px 20px; /* 增加内边距 */
  margin-bottom: 20px; /* 增加与第一条对话的距离 */
  font-weight: bold;
  color: #333;
  text-align: left;
  font-size: 18px; /* 增大字体大小 */
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding-top: 20px;
  padding-left: 10%;
  padding-right: 10%;
  background-color: transparent;
  /* 修改滚动条颜色 */
  scrollbar-width: thin;
  scrollbar-color: rgba(0, 105, 224, 0.3) transparent;
}

/* 为 Webkit 浏览器（如 Chrome、Safari）自定义滚动条样式 */
.chat-messages::-webkit-scrollbar {
  width: 6px;
}

.chat-messages::-webkit-scrollbar-track {
  background: transparent;
}

.chat-messages::-webkit-scrollbar-thumb {
  background-color: rgba(0, 105, 224, 0.3);
  border-radius: 3px;
}

.message {
  display: flex;
  margin-bottom: 20px;
}

.message .avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 10px;
  overflow: hidden;
}

.message .avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 50%;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #ffffff;
}

.message .content {
  background-color: rgba(255, 255, 255,1);
  padding: 12px 18px; /* 增加内边距 */
  border-radius: 10px;
  max-width: 80%;
  font-size: 16px; /* 增加字体大小 */
  line-height: 1.8; /* 增加行高 */
}

.message.user {
  flex-direction: row-reverse;
}

.message.user .avatar {
  margin-right: 0;
  margin-left: 10px;
}

.message.user .content {
  background-color: rgba(0, 105, 224, 0.12);
  color: black;
}

.input-area {
  padding: 20px 10% 0 10%;
  border-top: 0px solid #e0e0e0;
  background-color: transparent;
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

input {
  width: 100%;
  padding: 12px 110px 12px 50px; /* 调整右侧padding以适应新的按钮组 */
  border: 1px solid rgba(204, 204, 204, 0.5);
  border-radius: 25px;
  font-size: 16px;
  background-color: rgba(255, 255, 255, 0.7);
  transition: border-color 0.3s;
  height: 55px;
}

input:focus {
  outline: none;
  border-color: #0069e0;
}

input::placeholder {
  color: #969696;
}

.button-group {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
}

.input-icon {
  color: #0069e0;
  font-size: 24px;
  cursor: pointer;
}

.link-icon {
  position: absolute;
  left: 18px;
  top: 50%;
  transform: translateY(-50%);
}

.microphone-icon {
  margin-right: 0; /* 将右侧边距改为0 */
}

.separator {
  width: 1px;
  height: 25px;
  background-color: rgba(204, 204, 204, 0.5);
  margin: 0 10px;
}

.send-button {
  width: 40px;
  height: 40px;
  background: linear-gradient(to right, #0069e0, #0052bc); /* 保持一致的蓝色渐变 */
  border: none;
  color: white;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.send-button:disabled {
  background: rgba(0, 105, 224, 0.1);
  color: rgba(0, 82, 188, 0.3);
  cursor: default;
}

.send-button :deep(.el-icon) {
  font-size: 24px;
}

.send-button:not(:disabled):hover {
  opacity: 0.9;
}

/* 新增的免责声明样式 */
.disclaimer {
  font-size: 10px;
  color: #999;
  text-align: center;
  margin-top: 12px;
  margin-bottom: 12px;
}

.audio-wave {
  display: flex;
  align-items: center;
  height: 24px;
  width: 24px;
}

.audio-wave span {
  display: inline-block;
  width: 3px;
  height: 100%;
  margin-right: 1px;
  background: #0069e0;
  animation: audio-wave 0.8s infinite ease-in-out;
}

@keyframes audio-wave {
  0%, 100% { transform: scaleY(0.3); }
  50% { transform: scaleY(1); }
}

.message .content audio {
  margin-top: 10px;
  width: 100%;
}
</style>