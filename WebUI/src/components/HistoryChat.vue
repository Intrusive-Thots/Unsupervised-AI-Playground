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
import { computed, ref } from 'vue'

import ThumbnailPreviewStrip from './ThumbnailPreviewStrip.vue'
import {
  DropdownMenu,
  DropdownMenuTrigger,
  DropdownMenuContent,
  DropdownMenuItem,
} from '@/components/ui/dropdown-menu'
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog'
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { useConversations } from '@/assets/js/store/conversations'

// ⚡ Bolt Performance Optimization: Use a static empty array constant instead of inline array definitions inside loops
// Why: Returning an inline `[]` inside a loop or mapped function creates a new reference each time, causing unnecessary re-renders in children.
const EMPTY_ITEMS: { id: string; imageUrl: string }[] = []

// ⚡ Bolt Performance Optimization: Return a static empty array instead of inline `[]`
// Why: Prevents Vue from detecting a new array reference and unnecessarily re-rendering child components.
const EMPTY_IMAGES_ARRAY: { id: string; imageUrl: string }[] = []

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
          const images = (part.output as { images?: Array<{ imageUrl?: string }> }).images ?? []
          return images.map((img, imgIndex) => ({
            id: `${msgIndex}-${partIndex}-${imgIndex}`,
            imageUrl: img.imageUrl ?? '',
          }))
        }
        return EMPTY_IMAGES_ARRAY
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
</script>

<style scoped></style>
