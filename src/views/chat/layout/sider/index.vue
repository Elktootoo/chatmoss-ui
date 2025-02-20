<script setup lang='ts'>
import type { CSSProperties } from 'vue'
import { computed, ref, watch } from 'vue'
import { NButton, NCard, NDivider, NInput, NLayoutSider, NModal, NSelect, useDialog, useMessage } from 'naive-ui'
import Tips from '../../tips.vue'
import { useModel } from '../../components/Modal/hooks/useModal'
import List from './List.vue'
import { ShopInfo, exchangeOptions } from './data'
import Footer from './Footer.vue'
import PersonCenter from './../../components/PersonCenter.vue'
import Login from '@/views/login/index.vue'
import { useAppStore, useChatStore, useUserStore } from '@/store'
import { getToken } from '@/store/modules/auth/helper'
import { toMoss } from '@/api'
import { exchange } from '@/api/personCenter'
const person = ref(null) as any
const [registerModal, { openModel }] = useModel()
const userStore = useUserStore()
const showModal = ref(false)
const ms = useMessage()
const appStore = useAppStore()
const chatStore = useChatStore()
const token = ref(getToken())

// const { isMobile } = useBasicLayout()
const tab = ref('login')
const isMobile = ref(true)
const collapsed = computed(() => appStore.siderCollapsed)

function handleAdd() {
  chatStore.createChat()
}

function handleUpdateCollapsed() {
  appStore.setSiderCollapsed(!collapsed.value)
}

const getMobileClass = computed<CSSProperties>(() => {
  return {
    position: 'fixed',
    zIndex: 50,
  }
})

const mobileSafeArea = computed(() => {
  return {
    paddingBottom: 'env(safe-area-inset-bottom)',
  }
})

// 兑换字符数
const toMossCode = ref('')
async function toMossEvent() {
  if (toMossCode.value === '') {
    ms.error('请输入字符包兑换码，字符包点击下面继续购买，感谢您的支持', { duration: 5000 })
    return
  }
  try {
    await toMoss<any>({
      code: toMossCode.value,
    }) as any
    userStore.residueCountAPI()
    ms.info('兑换成功，感谢您的支持！', { duration: 5000 })
  }
  catch (error: any) {
    ms.error(error.msg, { duration: 5000 })
  }
}

// 设置内容
const showSettingModal = ref(false)

const exchangeMossCode = ref('')
function handleSettingSubmit() {
  showSettingModal.value = true
  exchangeMossCode.value = ''
  toMossCode.value = ''
}

// 个人中心
function myHomeSubmit() {
  openModel()
  person.value.updated()
}

function modifyPassword() {
  tab.value = 'forget'
  showModal.value = true
}
// 登录注册功能
function showModelEvent() {
  tab.value = 'login'
  showModal.value = true
}
function handleSubmit() {
  showModal.value = false
  userStore.residueCountAPI()
  token.value = getToken()
}

watch(
  isMobile,
  (val) => {
    appStore.setSiderCollapsed(val)
  },
  {
    immediate: true,
    flush: 'post',
  },
)

const personCenter = ref(ShopInfo)
const shopModal = ref(false)
const shopData = ref({
  title: '',
  shopImg: '',
})
function buyEvent(item: any) {
  shopModal.value = true
  shopData.value = item
}

const dialog = useDialog()
async function exchangeMossEvent() {
  if (!exchangeMossCode.value)
    return

  dialog.info({
    title: '兑换',
    content: '是否确定兑换此套餐？',
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: async () => {
      try {
        await exchange({ itemId: exchangeMossCode.value })
        userStore.residueCountAPI()
        ms.success('兑换成功', { duration: 5000 })
      }
      catch (error: any) {
        ms.error(error.msg, { duration: 5000 })
      }
    },
    onNegativeClick: () => {

    },
  })
}
</script>

<template>
  <NLayoutSider
    :collapsed="collapsed" :collapsed-width="0" :width="200" show-trigger="arrow-circle"
    collapse-mode="transform" position="absolute" bordered :style="getMobileClass"
    @update-collapsed="handleUpdateCollapsed"
  >
    <div class="flex flex-col h-full" :style="mobileSafeArea">
      <main class="flex flex-col flex-1 min-h-0">
        <div v-show="false" class="p-4">
          <NButton id="question-btn" dashed block @click="handleAdd">
            新建问题
          </NButton>
        </div>
        <div class="flex-1 min-h-0 pt-12 pb-12 overflow-hidden">
          <List />
        </div>
        <!-- 拓展功能区域 -->
        <div v-show="false" class="continuation">
          <div class="setting-main setting-main1" @click="myHomeSubmit">
            <!-- <img class="setting-btn" src="https://luomacode-1253302184.cos.ap-beijing.myqcloud.com/v2.0/icon2.png" alt=""> -->
            <div class="setting-text step6">
              个人中心
            </div>
          </div>
          <div class="setting-main setting-main2" @click="handleSettingSubmit">
            <!-- <img class="setting-btn" src="https://luomacode-1253302184.cos.ap-beijing.myqcloud.com/v2.0/icon3.png" alt=""> -->
            <div class="setting-text step5">
              ChatMoss商店
            </div>
          </div>
          <Tips @login="showModelEvent" />
        </div>
        <NModal
          v-model:show="showSettingModal" preset="card"
          style="min-width: 300px; height: 85vh; overflow: scroll; width: 80%;"
        >
          <NCard title="ChatMoss商店" :bordered="false" size="huge" role="dialog" aria-modal="true">
            <div>
              <div class="title-h1">
                兑换码（登录后才能兑换哦~）
              </div>
              <div class="flex">
                <NInput v-model:value="toMossCode" class="mr-2" type="text" placeholder="请输入您的兑换码" />
                <NButton type="primary" ghost @click="toMossEvent">
                  确定
                </NButton>
              </div>
            </div>
            <NDivider />

            <div>
              <div class="title-h1">
                字符兑换包月模式
              </div>
              <div class="flex">
                <NSelect
                  v-model:value="exchangeMossCode" :options="exchangeOptions" class="mr-2"
                  placeholder="请输入您的兑换码"
                />
                <NButton type="primary" ghost @click="exchangeMossEvent">
                  确定
                </NButton>
              </div>
            </div>
            <NDivider />
            <div class="">
              <h1 class="title-h1">
                chatMoss4.0包月商城
              </h1>
              <div class="tip-text-input1">
                小提示：不同的套餐次数可以累加
              </div>
              <div class="tip-text-input1" />
              <div class="flex flex-wrap">
                <div
                  v-for="(item, index) of personCenter.shopsV4" :key="index" :class="{ 'border-div': index === 0 }"
                  class="item m-2 border-gray-50 border rounded-lg divide-solid text-center flex items-center justify-center flex-wrap flex-col cursor-pointer"
                  @click="buyEvent(item)"
                >
                  <div class="title-h2">
                    {{ item.title }}
                  </div>
                  <div class="desc">
                    {{ item.desc }}
                  </div>
                  <div class="desc">
                    ￥ {{ item.count }}
                  </div>
                </div>
              </div>
            </div>
            <NDivider />
            <div class="">
              <h1 class="title-h1">
                chatMoss3.5包月商城
              </h1>
              <div class="tip-text-input1">
                小提示：不同的套餐次数可以累加
              </div>
              <div class="tip-text-input1" />
              <div class="flex flex-wrap">
                <div
                  v-for="(item, index) of personCenter.shopsV3" :key="index" :class="{ 'border-div': index === 0 }"
                  class="item m-2 border-gray-50 border rounded-lg divide-solid text-center flex items-center justify-center flex-wrap flex-col cursor-pointer"
                  @click="buyEvent(item)"
                >
                  <div class="title-h2">
                    {{ item.title }}
                  </div>
                  <div class="desc">
                    {{ item.desc }}
                  </div>
                  <div class="desc">
                    ￥ {{ item.count }}
                  </div>
                </div>
              </div>
            </div>
            <NDivider />
            <div class="">
              <h1 class="title-h1">
                字符包商城
              </h1>
              <!-- <div class="tip-text-input1">
                小提示：OpenAI限制了5美元key的速度，字符包速度不受影响（字符包用的是120美金的key）
              </div> -->
              <div class="flex flex-wrap">
                <div
                  v-for="(item, index) of personCenter.shops" :key="index" :class="{ 'border-div': index === 0 }"
                  class="item m-2 border-gray-50 border rounded-lg divide-solid text-center flex items-center justify-center flex-wrap flex-col cursor-pointer"
                  @click="buyEvent(item)"
                >
                  <div class="title-h2">
                    {{ item.title }}
                  </div>
                  <div class="desc">
                    {{ item.desc }}
                  </div>
                  <div class="desc">
                    ￥ {{ item.count }}
                  </div>
                </div>
              </div>
            </div>
            <NDivider />
          </NCard>
        </NModal>
        <!-- 购买字符数 -->
        <NModal v-model:show="shopModal">
          <NCard
            style="width: 400px" :title="shopData.title" size="huge" role="dialog" aria-modal="true"
            :mask-closable="true"
          >
            <div class="tip-text-input2">
              支付宝扫码购买（暂不支持微信）
            </div>
            <img class="shop-img" :src="shopData.shopImg" alt="">
            <div class="tip-text-input3">
              您的支持就是我们的动力，我们会持续迭代本软件，提供更多更方便的功能！
            </div>
          </NCard>
        </NModal>
        <!-- 登录注册功能 -->
        <NModal v-model:show="showModal" transform-origin="center">
          <NCard
            style="width:80%;max-width: 600px;" title="" :bordered="false" size="huge" role="dialog"
            aria-modal="true"
          >
            <Login :tab="tab" @loginSuccess="() => { handleSubmit() }" />
          </NCard>
        </NModal>
        <PersonCenter ref="person" @register="registerModal" @modify-password="modifyPassword" />
      </main>
      <Footer />
    </div>
  </NLayoutSider>
  <template v-if="isMobile">
    <div v-show="!collapsed" class="fixed inset-0 z-40 bg-black/40" @click="handleUpdateCollapsed" />
  </template>
</template>

<style lang="less">
.continuation {
  margin-bottom: 20px;
  margin-top: 5px;
}

.setting-main {
  width: 90%;
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 6px;
  border-radius: 6px;
  // background-color: #323232;
  margin: 0 auto;
  margin-bottom: 10px;

  &:active {
    transform: scale(.96);
  }

  &:hover {
    // background-color: #3c4250;
  }

  .setting-text {
    // color: rgba(232, 236, 239, 0.75);
    font-size: 10px;
  }

  .setting-btn {
    width: 20px;
    height: 20px;
    margin-left: 8px;
    margin-right: 12px;
  }
}

.title-h1 {
  margin: 10px 0px;
  color: #FF6666;
}

.title-h2 {
  color: #FF6666;
}

.tip-text-input1 {
  font-size: 12px;
  margin-top: 10px;
  margin-bottom: 10px;
}

.tip-text-input2 {
  font-size: 14px;
  margin-top: -10px;
  margin-bottom: 10px;
  color: #FF6666;
  text-align: center;
}

.tip-text-input3 {
  font-size: 12px;
  margin-top: 20px;
  text-align: center;
}

.shop-img {
  min-width: 260px;
  min-height: 260px;
  width: 260px;
  height: 260px;
  margin: 0 auto;
}

.n-card__content {}
</style>
