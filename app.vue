<template>
  <div class="bg-gray-900 min-h-screen text-white font-sans">
    
    <div v-if="!isAuthenticated" class="flex items-center justify-center min-h-screen p-4 bg-gray-900">
      <div class="bg-gray-800 p-8 rounded-2xl shadow-2xl max-w-sm w-full border border-gray-700 animate-fade-in">
        
        <div class="text-center mb-8">
          <span class="text-6xl drop-shadow-lg mb-4 block">🍿</span>
          <h2 class="text-2xl font-bold text-white tracking-wider">私人劇院</h2>
          <p class="text-gray-400 text-sm mt-2">Private Family Theater</p>
        </div>

        <div class="space-y-4">
          <input
            v-model="inputPassword"
            type="password"
            placeholder="請輸入訪問密碼"
            class="w-full bg-gray-900 border border-gray-600 rounded-lg p-4 text-white focus:outline-none focus:border-indigo-500 text-center tracking-[0.5em] font-bold text-lg"
            @keyup.enter="login"
          />
          
          <p v-if="errorMsg" class="text-red-400 text-sm text-center font-bold animate-shake">{{ errorMsg }}</p>

          <button 
            @click="login" 
            class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-4 rounded-lg transition-colors shadow-lg shadow-indigo-900/50"
          >
            解鎖進入
          </button>
        </div>
      </div>
    </div>

    <div v-else class="animate-fade-in">
      <NuxtPage />
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const isAuthenticated = ref(false)
const inputPassword = ref('')
const errorMsg = ref('')

// 網頁載入時，檢查瀏覽器是否已經有通行證
onMounted(() => {
  const savedAuth = localStorage.getItem('meowtube_visitor_auth')
  if (savedAuth === 'true') {
    isAuthenticated.value = true
  }
})

// 產生 YYMMDD 格式的當日密碼
const getDynamicPassword = () => {
  const today = new Date()
  const yy = String(today.getFullYear()).slice(-2)
  const mm = String(today.getMonth() + 1).padStart(2, '0')
  const dd = String(today.getDate()).padStart(2, '0')
  return `${yy}${mm}${dd}`
}

// 登入驗證邏輯
const login = () => {
  // 取得今天的正確密碼 (例如：2026年6月5日會回傳 '260605')
  const correctPassword = getDynamicPassword()
  
  if (inputPassword.value === correctPassword) {
    isAuthenticated.value = true
    errorMsg.value = ''
    // 將登入狀態存入瀏覽器，下次打開就不用重新輸入
    localStorage.setItem('meowtube_visitor_auth', 'true')
  } else {
    // 錯誤訊息保持一般化，絕對不提示是日期或動態密碼
    errorMsg.value = '❌ 密碼錯誤，請重新輸入'
    inputPassword.value = ''
  }
}
</script>

<style>
/* 全域深色背景，確保滑動時不會露出白底 */
body {
  background-color: #111827;
  margin: 0;
}

.animate-fade-in { animation: fadeIn 0.4s ease-out; }
@keyframes fadeIn { 
  from { opacity: 0; transform: translateY(10px); } 
  to { opacity: 1; transform: translateY(0); } 
}

.animate-shake { animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both; }
@keyframes shake {
  10%, 90% { transform: translate3d(-1px, 0, 0); }
  20%, 80% { transform: translate3d(2px, 0, 0); }
  30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
  40%, 60% { transform: translate3d(4px, 0, 0); }
}
</style>