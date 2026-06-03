<template>
  <div class="max-w-2xl mx-auto py-8">
    <h1 class="text-3xl font-bold mb-8 text-white">⚙️ 劇院管理後台 (直連模式)</h1>

    <div class="bg-gray-800 p-6 md:p-8 rounded-xl border border-gray-700 shadow-xl">
      <h2 class="text-xl font-semibold mb-6 text-red-500">上架新電影</h2>
      
      <div class="mb-6 p-4 bg-gray-900 rounded-lg border border-gray-600 text-sm text-gray-300">
        <p class="font-bold text-white mb-2">📌 上架教學：</p>
        <ol class="list-decimal pl-5 space-y-1">
          <li>請先將電影檔案拖曳上傳至你的 Telegram 群組。</li>
          <li>對著上傳好的影片按右鍵，選擇「複製訊息連結」。</li>
          <li>連結格式如 <code class="text-green-400">https://t.me/c/12345/105</code>，請將最後的數字 <code class="text-red-400">105</code> 填入下方欄位。</li>
        </ol>
      </div>

      <form @submit.prevent="handlePublish" class="space-y-6">
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2">電影名稱</label>
          <input 
            type="text" 
            v-model="form.title" 
            required 
            placeholder="例如：全面啟動"
            class="w-full bg-gray-900 border border-gray-700 rounded-md py-2 px-3 text-white focus:outline-none focus:border-red-500"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2">劇情簡介</label>
          <textarea 
            v-model="form.description" 
            rows="3" 
            placeholder="輸入電影簡介..."
            class="w-full bg-gray-900 border border-gray-700 rounded-md py-2 px-3 text-white focus:outline-none focus:border-red-500"
          ></textarea>
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2">海報圖片網址 (選填)</label>
          <input 
            type="url" 
            v-model="form.coverUrl" 
            placeholder="https://..."
            class="w-full bg-gray-900 border border-gray-700 rounded-md py-2 px-3 text-white focus:outline-none focus:border-red-500"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2">Telegram 訊息 ID (Message ID)</label>
          <input 
            type="number" 
            v-model="form.messageId" 
            required
            placeholder="例如：105"
            class="w-full bg-gray-900 border border-gray-700 rounded-md py-2 px-3 text-white focus:outline-none focus:border-red-500"
          />
        </div>

        <div class="pt-4 border-t border-gray-700">
          <button 
            type="submit" 
            :disabled="isPublishing"
            class="w-full bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-4 rounded-md transition-colors disabled:bg-gray-600 disabled:cursor-not-allowed"
          >
            {{ isPublishing ? '資料寫入中...' : '🚀 確認發布' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const supabase = useSupabaseClient()
const isPublishing = ref(false)

const form = reactive({
  title: '',
  description: '',
  coverUrl: '',
  messageId: ''
})

const handlePublish = async () => {
  if (!form.messageId) {
    alert('請填寫 Telegram Message ID！')
    return
  }

  try {
    isPublishing.value = true

    // 直接將資料寫入 Supabase 資料庫
    const { error: dbError } = await supabase
      .from('movies')
      .insert({
        title: form.title,
        description: form.description,
        cover_url: form.coverUrl,
        tg_message_id: parseInt(form.messageId)
      })

    if (dbError) throw new Error(`寫入資料庫失敗: ${dbError.message}`)

    alert('✅ 電影發布成功！回到首頁即可觀看。')
    
    // 清空表單
    form.title = ''
    form.description = ''
    form.coverUrl = ''
    form.messageId = ''

  } catch (error) {
    console.error(error)
    alert(`❌ 發生錯誤：\n${error.message}`)
  } finally {
    isPublishing.value = false
  }
}
</script>