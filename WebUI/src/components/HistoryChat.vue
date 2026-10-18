<template>
  <div class="flex flex-col space-y-2 pr-3 h-full overflow-y-auto">
    <HistoryChatItem
      v-for="key in reversedConversationKeys"
      :key="key"
      :conversationKey="key"
      :menuOpenKey="menuOpenKey"
      @updateMenuOpenKey="onUpdateMenuOpenKey"
      @conversationSelected="onConversationSelected"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import HistoryChatItem from './HistoryChatItem.vue'
import { useConversations } from '@/assets/js/store/conversations'

const EMPTY_ARRAY: never[] = []

const conversations = useConversations()

// ⚡ Bolt Performance Optimization: Prevent render thrashing by using static empty array reference
const EMPTY_IMAGES: { id: string; imageUrl: string }[] = []
const emits = defineEmits<{
  (e: 'conversationSelected'): void
}>()

const images = (conversation: AipgUiMessage[]) => {
  return conversation.flatMap((msg, msgIndex) =>
    msg.parts
      .filter(
        (part) =>
          (part.type === 'tool-comfyUI' || part.type === 'tool-comfyUiImageEdit') &&
          part.state === 'output-available',
      )
      .map((part, partIndex) => {
        if (
          (part.type === 'tool-comfyUI' || part.type === 'tool-comfyUiImageEdit') &&
          'output' in part &&
          part.output &&
          typeof part.output === 'object' &&
          'images' in part.output
        ) {
          const images = (part.output as { images?: Array<{ imageUrl?: string }> }).images ?? EMPTY_ARRAY
          return images.map((img, imgIndex) => ({
            id: `${msgIndex}-${partIndex}-${imgIndex}`,
            imageUrl: img.imageUrl ?? '',
          }))
        }
        return EMPTY_ARRAY
      })
      .flat()
      .filter(
        (img): img is { id: string; imageUrl: string } =>
          img !== null &&
          img !== undefined &&
          'imageUrl' in img &&
          typeof img.imageUrl === 'string' &&
          img.imageUrl.trim() !== '' &&
          'id' in img &&
          typeof img.id === 'string',
      ),
  )
}

const reversedConversationKeys = computed(() => {
  const list = conversations.conversationList ?? {}
  const keys = Object.keys(list).reverse()
  console.log('Reversed conversation keys:', list, keys)
  return keys
})

const menuOpenKey = ref<string | null>(null)

function onUpdateMenuOpenKey(key: string | null) {
  menuOpenKey.value = key
}

function onConversationSelected() {
  emits('conversationSelected')
}
</script>

<style scoped></style>
