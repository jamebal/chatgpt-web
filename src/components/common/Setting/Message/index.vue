<script setup lang='ts'>
import { ref } from 'vue'
import { NButton, NButtonGroup, NPopover, NSpace } from 'naive-ui'
import AvatarComponent from './Avatar.vue'
import TextComponent from './Text.vue'
import { t } from '@/locales'

interface Props {
  dateTime?: string
  model?: string
  text?: string
  inversion?: boolean
  error?: boolean
  loading?: boolean
  responseCount?: number
  usage?: {
    completion_tokens: number
    prompt_tokens: number
    total_tokens: number
    estimated: boolean
  }
}
const props = defineProps<Props>()

const emit = defineEmits<Emit>()

interface Emit {
  (ev: 'regenerate'): void
  (ev: 'delete'): void
  (ev: 'responseHistory', historyIndex: number): void
}

const textRef = ref<HTMLElement>()

const asRawText = ref(props.inversion)

const messageRef = ref<HTMLElement>()

const indexRef = ref<number>(0)
indexRef.value = props.responseCount ?? 0

const url_openai_token = 'https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them'

async function handlePreviousResponse(next: number) {
  if (indexRef.value + next < 1 || indexRef.value + next > props.responseCount!)
    return
  indexRef.value += next
  emit('responseHistory', indexRef.value - 1)
}
</script>

<template>
  <div
    ref="messageRef"
    class="flex w-full mb-6 overflow-hidden"
    :class="[{ 'flex-row-reverse': inversion }]"
  >
    <div
      class="flex items-center justify-center flex-shrink-0 h-8 overflow-hidden rounded-full basis-8"
      :class="[inversion ? 'ml-2' : 'mr-2']"
    >
      <AvatarComponent :image="inversion" />
    </div>
    <div class="overflow-hidden text-sm " :class="[inversion ? 'items-end' : 'items-start']">
      <p v-if="inversion" class="text-xs text-[#b4bbc4]" :class="[inversion ? 'text-right' : 'text-left']">
        {{ `${model ? (`${model} - `) : ''} ${new Date(dateTime as string).toLocaleString()}` }}
      </p>
      <p v-else class="text-xs text-[#b4bbc4]" :class="[inversion ? 'text-right' : 'text-left']">
        <NSpace>
          {{ new Date(dateTime as string).toLocaleString() }}
          <NButtonGroup v-if="!inversion && responseCount && responseCount > 1">
            <NButton
              style="cursor: pointer;"
              size="tiny" quaternary
              :disabled="indexRef === 1"
              @click="handlePreviousResponse(-1)"
            >
              <svg stroke="currentColor" fill="none" stroke-width="1.5" viewBox="-3 3 24 24" stroke-linecap="round" stroke-linejoin="round" class="h-3 w-3" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><polyline points="15 18 9 12 15 6" /></svg>
            </NButton>
            <span class="text-xs text-[#b4bbc4]"> {{ indexRef }} / {{ responseCount }}</span>
            <NButton
              style="cursor: pointer;"
              size="tiny" quaternary
              :disabled="indexRef === responseCount"
              @click="handlePreviousResponse(1)"
            >
              <svg stroke="currentColor" fill="none" stroke-width="1.5" viewBox="3 3 24 24" stroke-linecap="round" stroke-linejoin="round" class="h-3 w-3" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><polyline points="9 18 15 12 9 6" /></svg>
            </NButton>
          </NButtonGroup>
          <template v-if="usage">
            <NPopover trigger="hover">
              <template #trigger>
                <span>
                  <span>[</span>
                  <span>{{ usage.estimated ? '~' : '' }}</span>
                  <span>{{ usage.prompt_tokens }}+{{ usage.completion_tokens }}={{ usage.total_tokens }}</span>
                  <span>]</span>
                </span>
              </template>
              <span class="text-xs">
                {{ usage.estimated ? t('chat.usageEstimate') : '' }}
                {{ t('chat.usagePrompt') }} {{ usage.prompt_tokens }}
                + {{ t('chat.usageResponse') }} {{ usage.completion_tokens }}
                = {{ t('chat.usageTotal') }}<a :href="url_openai_token" target="_blank">(?)</a>
                {{ usage.total_tokens }}
              </span>
            </NPopover>
          </template>
        </NSpace>
      </p>
      <div
        class="flex items-end gap-1 mt-2"
        :class="[inversion ? 'flex-row-reverse' : 'flex-row']"
      >
        <TextComponent
          ref="textRef"
          :inversion="inversion"
          :error="error"
          :text="text"
          :loading="loading"
          :as-raw-text="asRawText"
        />
      </div>
    </div>
  </div>
</template>
