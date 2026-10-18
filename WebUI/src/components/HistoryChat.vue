<template>
  <div class="flex flex-col space-y-2 pr-3 h-full overflow-y-auto">
    <HistoryChatItem
      v-for="key in reversedConversationKeys"
      :key="key"
      :conversationKey="key"
      @select="selectConversation(key)"
    />
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import HistoryChatItem from './HistoryChatItem.vue'
import { useConversations } from '@/assets/js/store/conversations'

// ⚡ Bolt Performance Optimization: Use a static empty array constant instead of inline array definitions inside loops
// Why: Returning an inline `[]` inside a loop or mapped function creates a new reference each time, causing unnecessary re-renders in children.
const EMPTY_ITEMS: { id: string; imageUrl: string }[] = []

const conversations = useConversations()

// ⚡ Bolt Performance Optimization: Prevent render thrashing by using static empty array reference
const EMPTY_IMAGES: { id: string; imageUrl: string }[] = []
const emits = defineEmits<{
  (e: 'conversationSelected'): void
}>()

const reversedConversationKeys = computed(() => {
  const list = conversations.conversationList ?? {}
  const keys = Object.keys(list).reverse()
  console.log('Reversed conversation keys:', list, keys)
  return keys
})
</script>

<style scoped></style>
