<script setup>
import { ref } from 'vue'
import {
  Layout,
  Avatar,
  message,
  Input,
  Button
} from 'ant-design-vue'

const userMessage = ref('')
const messages = ref([
  {
    type: 'ai',
    content: '你好！我是AI助手，很高兴为你服务。'
  }
])

import { fetchEventSource } from '@microsoft/fetch-event-source'
import { nextTick } from 'vue'
import { marked } from 'marked'
import DOMPurify from 'dompurify'

const eventSource = ref(null)
const abortController = ref(null)

const handleSend = async () => {
  if (!userMessage.value) {
    message.warning('请输入内容')
    return
  }

  // 添加用户消息
  messages.value.push({
    type: 'user',
    content: userMessage.value
  })

  const question = userMessage.value
  userMessage.value = ''

  // 添加AI消息占位
  messages.value.push({
    type: 'ai',
    content: ''
  })

  try {
    // 如果存在之前的连接，先关闭
    if (eventSource.value) {
      eventSource.value.close()
      eventSource.value = null
    }
    if (abortController.value) {
      abortController.value.abort()
    }

    abortController.value = new AbortController()

    eventSource.value = fetchEventSource('http://localhost:7001/api/v1/chat/stream', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: question,
        sessionId: 0,
        modelConfigId: 0
      }),
      signal: abortController.value.signal,
      openWhenHidden: true,
      onmessage(event) {
        const res = event.data
        if (res !== '[DONE]' && res != null) {
          try {
            const data = JSON.parse(res)
            let content = data.content
            if (content != null) {
              const lastMessage = messages.value[messages.value.length - 1]
              if (content.includes('\n')) {
                // 处理包含换行的内容
                const text = lastMessage.content + content
                content = DOMPurify.sanitize(marked.parse(text))
                lastMessage.content = content
              } else {
                // 处理普通文本
                lastMessage.content += content
              }

              // 自动滚动到底部
              nextTick(() => {
                const messageList = document.querySelector('.message-list')
                if (messageList) {
                  messageList.scrollTop = messageList.scrollHeight
                }
              })
            }
          } catch (error) {
            console.error('解析消息失败:', error)
          }
        }
      },
      onclose() {
        console.log('连接已关闭')
        eventSource.value = null
        abortController.value = null
      },
      onerror(error) {
        console.error('连接错误:', error)
        message.error('连接出错，请重试')
        if (eventSource.value) {
          eventSource.value.close()
          eventSource.value = null
        }
        if (abortController.value) {
          abortController.value.abort()
          abortController.value = null
        }
      }
    })
  } catch (error) {
    message.error('发送消息失败')
    console.error('Error:', error)
  }
}
</script>

<template>
  <div class="chat-container">
    <div class="message-list">
      <div v-for="(msg, index) in messages" :key="index" :class="['message', msg.type === 'ai' ? 'ai-message' : 'user-message']">
        <template v-if="msg.type === 'ai'">
          <a-avatar class="message-avatar" src="https://api.dicebear.com/7.x/bottts/svg?seed=1" />
          <div class="message-content">
            {{ msg.content }}
          </div>
        </template>
        <template v-else>
          <div class="message-content user-content">
            {{ msg.content }}
          </div>
          <a-avatar class="message-avatar" src="https://api.dicebear.com/7.x/miniavs/svg?seed=1" />
        </template>
      </div>
    </div>
    <div class="input-area">
      <a-input
        v-model:value="userMessage"
        placeholder="输入你的问题..."
        :rows="3"
        type="textarea"
        @pressEnter.prevent="handleSend"
      />
      <a-button type="primary" @click="handleSend">发送</a-button>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.message-list {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 20px;
}

.ai-message {
  flex-direction: row;
}

.user-message {
  flex-direction: row-reverse;
}

.message-avatar {
  margin: 0 12px;
}

.message-content {
  background: #f0f2f5;
  padding: 12px 16px;
  border-radius: 12px;
  max-width: 800px;
}

.user-content {
  background: #1890ff;
  color: white;
}

.input-area {
  padding: 20px;
  background: #fff;
  border-top: 1px solid #f0f0f0;
  display: flex;
  gap: 12px;
}
</style>