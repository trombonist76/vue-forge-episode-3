<script setup lang="ts">
import { Message, User } from '~/types';
import { nanoid } from "nanoid";

interface ChatBoxContainer {
  scrollHeight: number;
  clientHeight: number;
  scrollTop: number;
}

const props = withDefaults(defineProps<{
  messages: Message[]
  users: User[]
  me: User
  usersTyping?: User[]
}>(),
  {
    usersTyping: () => []
  }
)


const emits = defineEmits<{
  (e: 'new-message', message: Message): void
}>()

const isOpen = ref<boolean>(false) 
const newMessage = ref<string>('')
const chatBoxContainer = ref<ChatBoxContainer | null>(null)
const toggleShow = () => {
  isOpen.value = true
}

const findUserById = (id: string): User => {
  const user = props.users.find(user => user.id === id)
  if (!user) throw new Error('User not found!')
  return user
}

const checkIsMe = (id: string): boolean => {
  const user = props.users.find(user => user.id === id)
  if (!user) throw new Error('User not found!')
  return props.me.id === id
}
const handleMessage = () => {
  const message = {
    id: nanoid(),
    userId: props.me.id,
    createdAt: new Date(new Date().getTime() - 2 * 60000),
    text: newMessage.value 
  }

  emits('new-message', message)
  newMessage.value = ''
}

const typingUser = computed(() => {
  if (props.usersTyping.length === 0) return
  const typings = props.usersTyping.map(user => user.name).join(' and ')
  return typings
})

watch(() => props.messages.length, async () => {
  if (chatBoxContainer.value === null) return
  await nextTick()
  const container = chatBoxContainer.value
  container.scrollTop = container.scrollHeight
})
</script>

<template>
  <div class="chatbox-wrapper fixed bottom-0 right-0 mr-6 mb-4">
    <button 
      v-if="!isOpen"
      @click="toggleShow"
      class="btn p-4 text-white bg-blue-500 h-16 w-16 hover:bg-blue-600">
      <IconChat></IconChat>
    </button>
    <div
      v-if="isOpen"
      class="chatbox flex flex-col w-96 max-h-[600px] bg-slate-700">
      <div class="header p-4 flex justify-between bg-slate-800">
        Customer Support Chat <span> > </span>
      </div>
      <div 
        class="content relative px-3 h-full flex flex-col gap-4 flex-1 justify-around pt-4 overflow-auto"
        ref="chatBoxContainer">
        <chat-bubble
          v-for="message in messages"
          :message="message"
          :user="findUserById(message.userId)"
          :isMe="checkIsMe(message.userId)"
        ></chat-bubble>
        <div class="mt-4 sticky bottom-0 left-0 w-full bg-slate-700">
          <input 
            type="text" 
            placeholder="Type your message" 
            class="input input-bordered p-2 w-full"
            v-model="newMessage"
            @keyup.enter="handleMessage" />
          <label class="label h-8">
            <span v-show="typingUser" class="label-text-alt"> {{ typingUser }} typing... </span>
          </label>
        </div>
      </div>
    </div>
  </div>
</template>
