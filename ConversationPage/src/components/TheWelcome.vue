<script setup>
import { ImageFilled, SendFilled } from '@vicons/material'

import { ref, onMounted, watch } from 'vue'
const question = ref('')
const conversationHistory = ref([
  { role: 'assistant', content: '欢迎咨询本产品，请输入你的问题。' },
])

const describeItems = ref([
  {
    thumbUrl: 'https://img.alicdn.com/imgextra/i3/764341610/O1CN01WUxDNI1NlQWh9MWuE_!!764341610-0-picasso.jpg',
    detialsUrl: 'https://staticfiles.hpstore.cn/NPI-RC/COMMERCIAL/7N9T8PC+7Z7K0PC+7Z7J9PC+7N9V7PC+7N9V8PC/img1.png',
  },
  {
    thumbUrl: 'https://gw.alicdn.com/imgextra/O1CN01iYY1YS2JHQJH9YI6j_!!2201222579396-0-picasso.jpg_Q75.jpg_.webp',
    detialsUrl: 'https://img.alicdn.com/imgextra/i2/133668489/O1CN01O5COo22Ca0zgO6znW_!!133668489.jpg',
  },
])

const selectedItemIndex = ref(0)
const conversationsScroll = ref(null)
const conversations = ref(null)

function onItemSelect(index) {
  selectedItemIndex.value = index
}
function handleInput(e) {
  question.value = e.target.innerHTML.replace(/(?:^(?:&nbsp;)+)|(?:(?:&nbsp;)+$)/g, '')
}
const sessionId = ref('')
onMounted(async () => {
  const response = await fetch('/v1/session/create')
  const data = response.json()
  sessionId.value = data.sessionId
})
function scrollToBottom() {
  setTimeout(() => {
    if (conversationsScroll.value && conversations.value) {
      conversationsScroll.value.scrollTo({ top: conversations.value.scrollHeight })
    }
  }, 1)
}
async function submit(e) {
  if (e.ctrlKey) {
    return
  }
  e.preventDefault()
  //TODO: ready to submit
  console.log(question.value)
  const payload = JSON.stringify({
    session_id: sessionId.value,
    message: question.value
  });
  conversationHistory.value.push({ role: 'user', content: question.value })
  e.target.innerHTML = ''
  question.value = ''
  const response = await fetch('/v1/conversation', {
    method: "POST",
    headers: {
      'Content-Type': 'text/event-stream'
    },
    body: payload,
  });
  if (!response.body) return;
  const reader = response.body
    .pipeThrough(new TextDecoderStream())
    .getReader();
  conversationHistory.value.push({ role: 'assistant', content: '...' })
  scrollToBottom()
  while (true) {

    const { value, done } = await reader.read();
    if (done) break;
    for (const v of value.split('\r\n\r\n')) {
      if (v === 'event: break') {
        conversationHistory.value.push({ role: 'assistant', content: '...' })
        scrollToBottom()
      } else if (v.startsWith('data: ')) {
        const arr = v.split('data: ')
        conversationHistory.value[conversationHistory.value.length - 1].content = arr[arr.length - 1]
      }
    }
  }
}
</script>
<style>
:root {
  --background-color: rgb(226, 236, 242);
  --divider-color: rgb(229, 230, 235);
  --background-color-darken1: color-mix(in srgb, var(--background-color), black 4%);
  --background-color-darken2: color-mix(in srgb, var(--background-color), white 6%);
}

.container {
  background-image: url('https://portal.volccdn.com/obj/volcfe/experience/index/bg-final.jpg');
  border: 0 !important;
  display: flex;
  align-items: center;
  height: 100vh;
  padding: 10px;
}

.window {
  border: 1px solid var(--background-color);
  border-radius: 10px;
  outline: 0;
  width: 600px;
  height: 100%;
  margin: 0 auto;
  background-color: var(--background-color);
  color: #222;
}

.window-header {
  padding: 5px;
  padding-top: 0px;
  border-bottom: 1px solid var(--divider-color);
  display: table;
  width: 100%;
}

.window-header>.window-title-bar {
  margin: 0;
  line-height: 1.428571429;
  display: table-cell;
  float: left !important;
  margin: 4px;
  font-weight: 800;
}

.chat-history {
  height: 400px;
  display: flex;
  flex-direction: column;
  border-bottom: 1px solid var(--divider-color);
}

.chat-bubble {
  margin: 10px;
  display: inline-block;
  position: relative;
  height: auto;
  padding: 5px;
  border-radius: 5px;
}

.chat-bubble-left {
  background-color: #eee;
  align-self: flex-start;
}

.chat-bubble-right {
  border: 1px solid rgb(55, 81, 111);
  color: rgb(55, 81, 111);
  align-self: flex-end;
}

.chat-user-panel {
  background-color: var(--background-color-darken1);
  padding: 5px 10px;
}

.chat-user-input-zone {
  position: relative;
  background-color: var(--background-color-darken2);
  height: 100%;
  padding: 5px 10px;
  border-radius: 10px;
}

.chat-user-input {
  height: 100px;
  max-height: 100px;
}

.chat-user-input-action {
  display: flex;
  justify-content: space-between;
}

[contenteditable]:focus {
  outline: 0px solid transparent;
}

[contenteditable='true']:empty:not(:focus):before {
  content: attr(data-ph);
  color: rgb(70, 70, 70);
}

.demo-item-list li {
  list-style-type: none;
  display: inline-grid;
  margin: 0 10px;
}

.demo-item-list li img {
  height: 140px;
  cursor: hand;
}

.demo-item-list li.selected {
  border: 2px solid blue;
}

.item-info img {
  width: 100%;
}

.item-side {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}
</style>
<template>
  <div class="container">
    <n-grid x-gap="12" :cols="2">
      <n-gi>
        <div class="item-side">
          <div class="item-info">
            <n-scrollbar style="max-height: 450px; height: 450px">
              <img :src="describeItems[selectedItemIndex].detialsUrl" />
            </n-scrollbar>
          </div>
          <div class="item-select">
            <ul class="demo-item-list">
              <li :class="{ 'selected': selectedItemIndex == i }" v-for="(item, i) in describeItems" v-bind:key="i"
                @click="onItemSelect(i)">
                <img :src="item.thumbUrl" />
              </li>
            </ul>
          </div>
        </div>
      </n-gi>
      <n-gi>
        <div class="window">
          <div class="window-header">
            <div class="window-title-bar">在线客服聊天</div>
          </div>
          <div class="window-content">
            <n-scrollbar style="padding: 10px; max-height: 400px; height: 400px" ref="conversationsScroll">
              <div class="chat-history" ref="conversations">
                <div :class="{
                  'chat-bubble': true,
                  'chat-bubble-right': message.role == 'user',
                  'chat-bubble-left': message.role == 'assistant'
                }" v-for="(message, i) of conversationHistory" v-bind:key="i">
                  {{ message.content }}
                </div>
              </div>
            </n-scrollbar>
            <div class="chat-user-panel">
              <div class="chat-user-toolbar">
                <n-button text style="font-size: 24px">
                  <n-icon>
                    <ImageFilled />
                  </n-icon>
                </n-button>
              </div>
              <div class="chat-user-input-zone">
                <n-scrollbar style="max-height: 100px">
                  <div class="chat-user-input" contenteditable="true" data-ph="请输入你的问题..." @input="handleInput"
                    @keydown.enter="submit"></div>
                </n-scrollbar>

                <div class="chat-user-input-action">
                  <div></div>
                  <n-button text style="font-size: 18px">
                    <n-icon>
                      <SendFilled @v-on:click="submit" />
                    </n-icon>
                  </n-button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </n-gi>
    </n-grid>
  </div>
</template>
