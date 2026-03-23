<script setup>
import { ref, onMounted } from "vue"

const name = ref("")
const brother = ref("")
const date = ref("")
const status = ref("欠席")
const message = ref("")
const history = ref([])
const todayStatus = ref("未送信")

const GAS_URL = "https://script.google.com/macros/s/AKfycbwDO9vI4vece_1h5s-IYPopKXW7c8I7_gNIQzhUf7z6nJrLN2dBlFiabj7ULaLh7HDd/exec"

// 今日の状態チェック
const checkToday = () => {
  const today = new Date().toISOString().split("T")[0]
  const todayData = history.value.find(h => h.date === today)

  if (todayData) {
    todayStatus.value = `${todayData.status}（送信済）`
  } else {
    todayStatus.value = "未送信"
  }
}

// 初期処理
onMounted(() => {
  name.value = localStorage.getItem("name") || ""
  brother.value = localStorage.getItem("brother") || ""

  const today = new Date().toISOString().split("T")[0]
  date.value = today

  const saved = localStorage.getItem("history")
  if (saved) {
    history.value = JSON.parse(saved)
  }

  checkToday()
})

// 履歴保存
const saveHistory = (data) => {
  history.value = history.value.filter(h => h.date !== data.date)
  history.value.unshift(data)
  localStorage.setItem("history", JSON.stringify(history.value))
}

// 送信
const sendData = async () => {
  if (!name.value) {
    alert("名前を入力してください")
    return
  }

  const data = {
    id: Date.now(),
    name: name.value,
    brother: brother.value,
    date: date.value,
    status: status.value
  }

  saveHistory(data)

  localStorage.setItem("name", name.value)
  localStorage.setItem("brother", brother.value)

  checkToday()

  try {
    await fetch(GAS_URL, {
      method: "POST",
      body: JSON.stringify(data)
    })

    message.value = "送信完了しました！"
  } catch {
    message.value = "送信失敗"
  }
}

// 削除
const remove = (id) => {
  history.value = history.value.filter(h => h.id !== id)
  localStorage.setItem("history", JSON.stringify(history.value))
  checkToday()
}

// 編集
const edit = (item) => {
  name.value = item.name
  brother.value = item.brother
  date.value = item.date
  status.value = item.status
}
</script>

<template>
  <div class="container">
    <h1>出欠連絡</h1>

    <!-- 今日の状態 -->
    <h2>今日の状態</h2>
    <div 
      class="today-box"
      :class="{
        red: todayStatus.includes('欠席'),
        blue: todayStatus.includes('10時'),
        gray: todayStatus === '未送信'
      }"
    >
      {{ todayStatus }}
    </div>

    <!-- 入力フォーム -->
    <label>名前</label>
    <input v-model="name" type="text" />

    <label>兄弟（任意）</label>
    <input v-model="brother" type="text" />

    <label>日付</label>
    <input v-model="date" type="date" />

    <label>状態</label>
    <div class="btn-group">
      <button 
        :class="['btn', status === '欠席' ? 'active-red' : '']"
        @click="status = '欠席'"
      >
        欠席
      </button>

      <button 
        :class="['btn', status === '10時以降参加' ? 'active-blue' : '']"
        @click="status = '10時以降参加'"
      >
        10時参加
      </button>
    </div>

    <button class="send-btn" @click="sendData">送信</button>

    <p class="msg">{{ message }}</p>

    <!-- 履歴 -->
    <h2>送信履歴</h2>

    <div 
      v-for="item in history" 
      :key="item.id" 
      class="history-item"
    >
      <div>
        <strong>{{ item.date }}</strong><br>
        {{ item.status }}<br>

        <small class="name-text">
          {{ item.name }}
          <span v-if="item.brother"> / {{ item.brother }}</span>
        </small>
      </div>

      <div class="actions">
        <button @click="edit(item)">✏️</button>
        <button @click="remove(item.id)">🗑</button>
      </div>
    </div>
  </div>
</template>

<style>
.container {
  max-width: 400px;
  margin: auto;
  padding: 20px;
  font-family: sans-serif;
}

input {
  width: 100%;
  padding: 10px;
  margin-top: 10px;
  font-size: 16px;
}

.btn-group {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.btn {
  flex: 1;
  padding: 12px;
  border: none;
  border-radius: 10px;
  background: #ddd;
  font-size: 16px;
}

.active-red {
  background: #ff6b6b;
  color: white;
}

.active-blue {
  background: #4dabf7;
  color: white;
}

.send-btn {
  width: 100%;
  padding: 15px;
  margin-top: 15px;
  background: #51cf66;
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 18px;
}

.msg {
  text-align: center;
  margin-top: 10px;
}

/* 今日の状態 */
.today-box {
  padding: 12px;
  margin-bottom: 15px;
  border-radius: 10px;
  text-align: center;
  font-weight: bold;
}

.red {
  background: #ffe3e3;
  color: #c92a2a;
}

.blue {
  background: #e7f5ff;
  color: #1971c2;
}

.gray {
  background: #f1f3f5;
  color: #495057;
}

.history-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 10px;
}

.actions button {
  margin-left: 5px;
}

.name-text {
  color: #666;
  font-size: 12px;
}
</style>