<template>
  <div class="login-page">
    <div class="background">
      <div class="glow"></div>
    </div>
    <div class="content">
      <div class="center-content">
        <h1>为什么不问问神奇海螺呢？</h1>
        <p>关于蟹堡王和汉堡制作的一切知识</p>
        <button class="magic-button" @click="goToMainPage">
          <span>召唤神奇海螺</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router';

const router = useRouter();

const goToMainPage = () => {
  router.push('/main');
};
</script>

<style scoped>
.login-page {
  position: relative;
  height: 100vh;
  overflow: hidden;
  background: url('@/assets/images/backGround.jpg') no-repeat center center;
  background-size: cover; /* 这将确保图片覆盖整个区域且不变形 */
}

.background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  background: rgba(10, 25, 47, 0.7); /* 添加半透明遮罩，使背景不那么刺眼 */
}

.glow {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: radial-gradient(
    ellipse at center,
    rgba(29, 78, 216, 0.15) 0%,
    transparent 70%
  );
  filter: blur(60px);
  opacity: 0.8;
  animation: pulse 10s ease-in-out infinite alternate;
}

@keyframes pulse {
  0%, 100% {
    opacity: 0.5;
  }
  50% {
    opacity: 1;
  }
}

.content {
  position: relative;
  display: flex;
  height: 100%;
  z-index: 1;
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}

.center-content {
  text-align: center;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

h1 {
  font-size: 4em;
  margin-bottom: 20px;
  letter-spacing: 2px;
  text-shadow: 0 0 10px rgba(65, 184, 255, 0.7);
  animation: glow 3s ease-in-out infinite alternate;
  margin-left: 0;
  margin-right: 0;
}

@keyframes glow {
  from {
    text-shadow: 0 0 10px rgba(65, 184, 255, 0.7);
  }
  to {
    text-shadow: 0 0 20px rgba(65, 184, 255, 0.9), 0 0 30px rgba(65, 184, 255, 0.7);
  }
}

p {
  font-size: 1.4em;
  text-align: center;
  color: rgba(255, 255, 255, 0.8);
  text-shadow: 0 0 5px rgba(65, 184, 255, 0.5);
  margin: 0;
}

.magic-button {
  margin-top: 40px;
  padding: 15px 30px;
  font-size: 1.2em;
  color: #fff;
  background: linear-gradient(90deg, #00d2ff, #3a7bd5, #00d2ff);
  background-size: 200% 100%;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 0 15px rgba(0, 210, 255, 0.3);
}

.magic-button::before {
  content: '';
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  background: linear-gradient(45deg, #00d2ff, #3a7bd5, #2ecc71, #00d2ff);
  background-size: 400% 400%;
  z-index: -1;
  animation: glowing 3s ease-in-out infinite;
  border-radius: 25px;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.magic-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 0 20px rgba(0, 210, 255, 0.5);
}

.magic-button:hover::before {
  opacity: 1;
}

.magic-button:hover {
  background-position: 100% 0;
}

.magic-button:active {
  transform: translateY(1px);
}

@keyframes glowing {
  0% {
    background-position: 0 0;
  }
  50% {
    background-position: 400% 0;
  }
  100% {
    background-position: 0 0;
  }
}
</style>
