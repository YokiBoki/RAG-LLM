<template>
  <div class="ai-practice-container">
    <!-- 左侧历史对话记录 -->
    <div class="history-panel">
      <div class="new-chat-container">
        <button class="new-chat-btn" @click="newConversation">
          新建对话
          <el-icon class="plus-icon">
            <Plus/>
          </el-icon>
        </button>
      </div>
      <ul class="history-list">
        <li v-for="(item, index) in historyList" :key="index" @click="selectConversation(index)"
            :class="{ active: currentConversationIndex === index }">
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
                <img src="@/assets/images/robot.png" alt="AI Avatar">
              </div>
              <div v-else>
                <img src="@/assets/images/user.png" alt="Me">
              </div>
            </div>
            <div class="content">
              {{ message.content }}
              <span v-if="message.role === 'assistant'" class="play-icon" @click="playVoice(message.content)">🔊</span>
            </div>
          </div>
        </div>

        <!-- 输入框 -->
        <div class="input-area">
          <div class="input-wrapper">
            <el-icon class="input-icon link-icon">
              <Link/>
            </el-icon>
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
                <Microphone/>
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
                    <el-icon>
                      <Top/>
                    </el-icon>
                  </el-button>
                </template>
                <span>请文字/录音/上传语音回复</span>
              </el-popover>
            </div>
          </div>
        </div>

        <div class="disclaimer">
          服务生成的所有内容均由AI生成
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref, computed, nextTick, onMounted, onUnmounted} from 'vue';
import {Link, Microphone} from '@element-plus/icons-vue';
import {ElMessage} from 'element-plus';
// import AudioBase from "@/components/AudioBase.vue";
import {get, post} from '@/utils/request'
import {API} from '@/api/config'
import '@/assets/chatpage.css'

const playingText = ref('');
const ttsCache = new Map();
const playVoice = async (text) => {

  if (ttsCache.has(text)) {
    const cachedUrl = ttsCache.get(text);
    new Audio(cachedUrl).play();
    return;
  }

  try {
    playingText.value = text;  // 标记为当前播放中
    const response = await fetch(`http://localhost:8000/api/tts?text=${encodeURIComponent(text)}&rate=-10%`);
    const audioBlob = await response.blob();
    const audioUrl = URL.createObjectURL(audioBlob);
    const audio = new Audio(audioUrl);
    audio.play();
    audio.onended = () => playingText.value = '';
  } catch (error) {
    ElMessage.error('播放失败，请稍后重试');
    playingText.value = '';
  }
};

const historyList = ref([
  {
    title: '健康知识咨询',
    messages: [
      {role: 'assistant', content: '您好！我是您的健康助手，你需要咨询什么健康问题？'},
      {role: 'user', content: '我最近总是感到疲劳，请问这是怎么回事？'},
      {
        role: 'assistant',
        content: '别担心！让我来帮您分析一下。疲劳可能是由于多种原因引起的，包括睡眠不足、营养不良、缺乏运动等。建议您保持良好的作息习惯，均衡饮食，适量运动。如果症状持续，建议咨询医生进行进一步检查。'
      }
    ]
  },

  
]);

const currentConversationIndex = ref(0);
const userInput = ref('');
const isListening = ref(false);
const chatMessagesRef = ref(null);
const isRecording = ref(false);
let mediaRecorder = null;
let audioChunks = [];
let recognition = null;
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
    // 实际上这里要和后端交互的话，messages 最好用 map 格式，key 是对应的 id，这样方便后端根据 id 来操作消息
    messages: [{
      role: 'assistant',
      content: '您好！我是您的健康助手，你需要咨询什么健康问题？',
    }]
  });
  currentConversationIndex.value = 0;
};

const sendMessage = async () => {
  if (userInput.value.trim()) {
    // 添加用户消息
    currentConversation.value.messages.push({role: 'user', content: userInput.value});
    const prompt = userInput.value;
    userInput.value = '';
    nextTick(() => {
      scrollToBottom();
    });

    const loadingMessage = ref({
      role: 'assistant',
      content: '正在思考...',
      loading: true // 标记为加载状态
    });
    currentConversation.value.messages.push(loadingMessage.value);
    nextTick(() => {
      scrollToBottom();
    });

    // 获取AI回复
    try {
      const res = await get(API.GENERATE, {prompt: prompt});
      if (res.code == 100) {
        // 更新加载中的消息为实际回复
        loadingMessage.value.content = res.data;
        loadingMessage.value.loading = false; // 更新为非加载状态
        nextTick(() => {
          scrollToBottom();
        });
      } else {
        ElMessage.error(res.msg);
        loadingMessage.value.content = '获取回复失败，请稍后重试';
        loadingMessage.value.loading = false;
        nextTick(() => {
          scrollToBottom();
        });
      }
    } catch (error) {
      console.error('sendMessage error', error);
      ElMessage.error('获取回复失败，请稍后重试');
      loadingMessage.value.content = '获取回复失败，请稍后重试';
      loadingMessage.value.loading = false;
      nextTick(() => {
        scrollToBottom();
      });
    }
  }
};

const finishRecording = () => {
    stopVoiceRecognition();
    isInputDisabled.value = false;
  
};

const stopVoiceRecognition = () => {
  if (recognition) {
    recognition.stop(); // 这会自动触发 onend 事件
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

const toggleRecording = () => {
  if (!isRecording.value) {
    startVoiceRecognition();
  } else {
    stopVoiceRecognition();
  }
  isRecording.value = !isRecording.value;
};

const startVoiceRecognition = () => {
  recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
  recognition.lang = 'zh-CN';
  recognition.continuous = false;
  recognition.interimResults = false;

  recognition.onstart = () => {
    isInputDisabled.value = true; // 语音输入时禁用文本框
    isRecording.value = true;
  };

  recognition.onresult = (event) => {
    const transcript = event.results[0][0].transcript;
    userInput.value = transcript;
    sendMessage();
  };

  recognition.onerror = () => {
    isRecording.value = false;
  };

  recognition.onend = () => {
    isRecording.value = false;
    isInputDisabled.value = false; // 恢复文本框
  };

  recognition.start();
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

onMounted(() => {
});

onUnmounted(() => {
  finishRecording();
  stopMediaStream();
});
</script>
