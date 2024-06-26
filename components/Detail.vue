<script setup lang="ts">
import { useClipboard } from '@vueuse/core'
const props = defineProps({
  emoji: {
    type: [Object],
    default: null
  }
})
const route = useRoute()
const { locale, t } = useI18n()
const rtl = computed(() => ['ar', 'he'].includes(locale.value))

const defaultData = {
  c: props.emoji?.c || '',
  q: '',
  e: props.emoji?.e || '',
  v: '',
  n: '',
  g: props.emoji?.g || '',
  s: props.emoji?.s || '',
  t: props.emoji?.t || '',
  k: []
}
const id = props.emoji?.c || route.params.id
const { data: fetchData, error } = useFetch(`/api/emoji/${id}`, { query: { locale: locale.value } })
const data = computed(() => fetchData.value || defaultData)

const title = computed(() => t('seo.detailTitle', { emoji: data.value.e, name: data.value.t }))
const description = computed(() =>
  t('seo.detailDescription', {
    emoji: data.value.e,
    name: data.value.t,
    version: data.value.v,
    group: data.value.g,
    code: `U+${data.value.c.replace(/ /g, ' U+')}`,
    oName: data.value.n
  })
)
useHead({
  title,
  meta: [{ name: 'description', content: description }]
})
const source = computed(() => data.value.e)
const { copy, copied } = useClipboard({ source, legacy: true })
function handleCopy (e: KeyboardEvent) {
  if ((e.metaKey || e.ctrlKey) && e.key === 'c') {
    const selection = document.getSelection()
    if (!selection?.toString()) {
      copy(source.value)
    }
  }
}
const isMac = ref(false)
const isAndroidQQ = ref(false)
onMounted(() => {
  document.addEventListener('keydown', handleCopy)
  isMac.value = window.navigator.userAgent.toLowerCase().includes('macintosh')
  isAndroidQQ.value =
    window.navigator.userAgent.toLowerCase().includes('android') &&
    (window.navigator.userAgent.toLowerCase().includes('mqq') || window.navigator.userAgent.toLowerCase().includes('micromessenger'))
})
onUnmounted(() => {
  document.removeEventListener('keydown', handleCopy)
})
const platform = [
  {
    name: 'Apple',
    imgPath: 'apple'
  },
  {
    name: 'Google',
    imgPath: 'google'
  },
  {
    name: 'Facebook',
    imgPath: 'facebook'
  },
  {
    name: 'X / Twitter',
    imgPath: 'x'
  },
  {
    name: 'Microsoft',
    imgPath: 'microsoft'
  },
  {
    name: 'Samsung',
    imgPath: 'samsung'
  },
  {
    name: 'Whatsapp',
    imgPath: 'whatsapp'
  }
]
const localePath = useLocalePath()
const schema = computed(() => [
  // delete when nuxt-schema-org fix the bug, delete defineWebSite and defineWebPage
  // defineWebSite({
  //   '@id': 'https://searchemoji.app/#website',
  //   '@type': 'WebSite',
  //   description: t('seo.title'),
  //   inLanguage: locale.value,
  //   name: 'SearchEmoji',
  //   url: 'https://searchemoji.app',
  //   publisher: {
  //     '@id': 'https://searchemoji.app/#identity'
  //   }
  // }),
  // defineWebPage({
  //   '@id': `https://searchemoji.app${localePath('/' + data.value?.c || '')}/#webpage`,
  //   '@type': 'WebPage',
  //   description: description.value,
  //   name: title.value,
  //   url: localePath('/' + data.value?.c || ''),
  //   about: {
  //     '@id': 'https://searchemoji.app/#identity'
  //   },
  //   isPartOf: {
  //     '@id': 'https://searchemoji.app/#website'
  //   }
  // }),
  {
    '@id': `https://searchemoji.app${localePath('/' + data.value?.c || '')}/#VisualArtwork`,
    '@type': 'VisualArtwork',
    name: data.value?.t || '',
    alternateName: data.value?.n || '',
    keywords: data.value?.k.join(',') || '',
    url: localePath('/' + data.value?.c || ''),
    creator: {
      '@type': 'Organization',
      name: 'Unicode',
      url: 'https://unicode.org/'
    },
    image: {
      '@type': 'ImageObject',
      contentUrl: `https://img.searchemoji.app/emoji-images/apple/${data.value?.c.toLowerCase()}.webp`,
      width: 144,
      height: 144
    },
    version: data.value?.v || ''
  }
])
useSchemaOrg(schema)
</script>

<template>
  <div class="max-w-[680px] mx-auto my-8 px-4">
    <div v-if="error" class="text-rose-500 card p-6 mb-6 rounded-2xl">
      {{ error }}
      <button class="border border-rose-500 px-2 rounded-full" onclick="window.location.reload()">{{ $t('refresh') }}</button>
    </div>
    <main v-if="data">
      <div class="flex justify-center items-center mx-auto">
        <h2 class="text-[128px]">{{ data.e }}</h2>
      </div>
      <div>
        <h3 class="font-bold text-2xl color-title text-center">{{ data.t }}</h3>
        <button
          class="px-6 h-12 rounded-2xl bg-rose-500 text-white flex items-center mx-auto my-6 md:tooltip"
          :class="rtl ? 'flex-row-reverse' : ''"
          :data-tip="copied ? $t('copied') : `${isMac ? '⌘' : 'Ctrl'} + C`"
          @click="copy(source)"
        >
          <i
            class="text-xl"
            :class="[copied ? 'icon-[material-symbols--check-circle] text-success' : 'icon-[material-symbols--content-copy-outline]', rtl ? 'ml-2' : 'mr-2']"
            aria-hidden="true"
            role="img"
          />
          {{ $t('copyBtn') }}
        </button>
        <p v-if="isAndroidQQ && locale === 'zh-hans'" class="text-center color-disable text-sm mb-4">
          点击复制功能在安卓微信或 QQ 浏览器中可能会失效，如果不能复制，可以长按上面的 Emoji 手动复制。或者点击右上角，在浏览器中打开使用本站。
        </p>
        <div class="flex justify-between border-t border-b border-main py-2" :class="rtl ? 'flex-row-reverse' : ''">
          <span class="shrink-0" :class="rtl ? 'flex-row-reverse ml-4' : 'mr-4'">{{ $t('unicodeName') }}</span> <span>{{ data.n }}</span>
        </div>
        <div class="flex justify-between border-b border-main py-2" :class="rtl ? 'flex-row-reverse' : ''">
          <span class="shrink-0 my-0.5" :class="rtl ? 'flex-row-reverse ml-4' : 'mr-4'">{{ $t('searchKeyword') }}</span>
          <div class="flex items-center flex-wrap justify-end" :class="rtl ? 'flex-row-reverse' : ''">
            <span v-for="k in data.k" :key="k" class="card color-action rounded-xl px-2 my-0.5" :class="rtl ? 'flex-row-reverse mr-1' : 'ml-1'">{{ k }}</span>
          </div>
        </div>
        <div class="flex justify-between border-b border-main py-2" :class="rtl ? 'flex-row-reverse' : ''">
          <span class="shrink-0" :class="rtl ? 'flex-row-reverse ml-4' : 'mr-4'">{{ $t('version') }}</span> <span>{{ data.v }}</span>
        </div>
        <div class="flex justify-between border-b border-main py-2" :class="rtl ? 'flex-row-reverse' : ''">
          <span class="shrink-0" :class="rtl ? 'flex-row-reverse ml-4' : 'mr-4'">{{ $t('code') }}</span> <span>U+{{ data.c.replace(/ /g, ' U+') }}</span>
        </div>
        <div class="flex justify-between border-b border-main py-2" :class="rtl ? 'flex-row-reverse' : ''">
          <span class="shrink-0" :class="rtl ? 'flex-row-reverse ml-4' : 'mr-4'">{{ $t('inGroup') }}</span>
          <span class="text-right">{{ data.g }} > {{ data.s }}</span>
        </div>
      </div>
      <div class="mt-6">
        <h4 class="font-bold color-action" :class="rtl ? 'text-right' : ''">{{ $t('otherPlatform') }}</h4>
        <div
          class="mt-6 flex justify-between md:block bg-white/90 dark:bg-zinc-800/90 border border-zinc-200/80 dark:border-zinc-700/80 p-4 md:p-0 md:bg-transparent md:dark:bg-transparent md:border-none rounded-2xl"
        >
          <div class="md:flex md:border-t md:border-b md:border-main md:py-2">
            <div v-for="p in platform" :key="p.imgPath" class="md:flex-1 md:text-center h-[72px] leading-[72px] md:h-auto md:leading-normal mb-2 md:mb-0">
              {{ p.name }}
            </div>
          </div>
          <div class="md:flex md:mt-2">
            <div v-for="p in platform" :key="p.imgPath" class="mb-2 md:flex-1 md:flex md:justify-center md:tooltip" :data-tip="$t('imgCopyTip')">
              <img
                :src="`https://img.searchemoji.app/emoji-images/${p.imgPath}/${data.c.toLowerCase()}.webp`"
                width="72"
                height="72"
                :alt="$t('platformImg', { name: data.n, platform: p.name })"
              >
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped></style>
