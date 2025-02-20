<script setup lang='ts'>
import { computed } from 'vue'
import { NInput, NPopconfirm, NScrollbar } from 'naive-ui'
import { SvgIcon } from '@/components/common'
import { useAppStore, useChatStore } from '@/store'
import { useBasicLayout } from '@/hooks/useBasicLayout'

const { isMobile } = useBasicLayout()

const appStore = useAppStore()
const chatStore = useChatStore()

// console.log(chatStore.$state)
const dataSources = computed(() => chatStore.chatsCollect)

async function handleSelect({ id }: Chat.ChatInfo) {
  if (isActive(id))
    return
  await chatStore.setActive(id)

  if (chatStore.active)
    chatStore.updateHistory(id, { isEdit: false })

  if (isMobile.value)
    appStore.setSiderCollapsed(true)
}

function handleEdit({ id }: Chat.ChatInfo, isEdit: boolean, event?: MouseEvent) {
  event?.stopPropagation()
  chatStore.updateHistory(id, { isEdit })
}

function handleDelete(index: number, event?: MouseEvent | TouchEvent) {
  event?.stopPropagation()
  chatStore.deleteHistory(index)
}

function handleEnter({ id }: Chat.ChatInfo, isEdit: boolean, event: KeyboardEvent) {
  event?.stopPropagation()
  if (event.key === 'Enter')
    chatStore.updateHistory(id, { isEdit })
}

function isActive(id: any) {
  return chatStore.active === id
}
</script>

<template>
  <NScrollbar class="px-4">
    <div class="flex flex-col gap-2 text-sm">
      <template v-if="!dataSources.length">
        <div class="flex flex-col items-center mt-4 text-center text-neutral-300">
          <SvgIcon icon="ri:inbox-line" class="mb-2 text-3xl" />
          <span>{{ $t('common.noData') }}</span>
        </div>
      </template>
      <template v-else>
        <div v-for="(item, index) of dataSources" :key="index" class="group">
          <a
            class="question-list relative flex items-center gap-3 px-3 py-3 break-all border rounded-md cursor-pointer hover:bg-neutral-100 group dark:border-neutral-800 dark:hover:bg-[#24272e]"
            :class="isActive(item.id) && ['border-[#00CCFF]', 'bg-neutral-100', 'text-[#0099FF]', 'dark:bg-[#24272e]', 'dark:border-[#0099FF]', 'pr-14']"
            @click="handleSelect(item)"
          >
            <span>
              <SvgIcon icon="ri:message-3-line" />
            </span>
            <div class="relative flex-1 overflow-hidden break-all text-ellipsis whitespace-nowrap">
              <NInput
                v-if="item.isEdit"
                v-model:value="item.tem"
                size="tiny"
                @keypress="handleEnter(item, false, $event)"
              />
              <span v-else>{{ item.title || '新建问题' }}</span>
            </div>
            <!-- v-if="isActive(item.uuid)" -->
            <div class="absolute z-10 flex visible right-1">
              <template v-if="item.isEdit && isActive(item.id)">
                <button class="p-1" @click="handleEdit(item, false, $event)">
                  <SvgIcon icon="ri:save-line" />
                </button>
              </template>
              <template v-if="!item.isEdit">
                <button v-if="isActive(item.id)" class="p-1">
                  <SvgIcon icon="ri:edit-line" @click="handleEdit(item, true, $event)" />
                </button>
                <!-- group-hover:visible -->
                <div :class="isActive(item.id) ? 'visible' : 'invisible group-hover:visible'">
                  <NPopconfirm placement="bottom" @positive-click="handleDelete(index, $event)">
                    <template #trigger>
                      <button class="p-1" @click.stop>
                        <SvgIcon icon="ri:delete-bin-line" />
                      </button>
                    </template>
                    {{ $t('chat.deleteHistoryConfirm') }}
                  </NPopconfirm>
                </div>
              </template>
            </div>
          </a>
        </div>
      </template>
    </div>
  </NScrollbar>
</template>
