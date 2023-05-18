<script setup lang="ts">
import { Message, User } from '~/types'
import { useTimeAgo } from '@vueuse/core'
import Markdown from 'vue3-markdown-it'

const timeAgo = useTimeAgo(new Date(2021, 0, 1))
 const props = defineProps<{
	message?: Message
	user: User
	isMe: boolean
 }>()

const messageTime = computed(() => props.message && useTimeAgo(props.message.createdAt).value.replaceAll('"',''))
</script>

<template>
<div :class="['chat', isMe ? 'chat-end' : 'chat-start']">	
  <div class="chat-image avatar">
    <div class="w-10 rounded-full">
      <img :src="props.user.avatar" alt="user"/>
    </div>
  </div>
  <div class="chat-header flex gap-2 items-center px-2 mb-1">
		{{ user.name }}
    <time v-if="message" class="text-xs opacity-50">{{ messageTime }}</time>
  </div>
  <div  class="chat-bubble">
    <Markdown v-if="message" :source="message.text"/>
    <AppLoading v-else/>
  </div>
</div>
</template>
