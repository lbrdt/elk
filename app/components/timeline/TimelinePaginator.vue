<script setup lang="ts">
import type { mastodon } from 'masto'
// @ts-expect-error missing types
import { DynamicScrollerItem } from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

const { account, buffer = 10, endMessage = true, followedTags = [] } = defineProps<{
  paginator: mastodon.Paginator<mastodon.v1.Status[], mastodon.rest.v1.ListAccountStatusesParams>
  stream?: mastodon.streaming.Subscription
  context?: mastodon.v2.FilterContext
  account?: mastodon.v1.Account
  followedTags?: mastodon.v1.Tag[]
  preprocess?: (items: mastodon.v1.Status[]) => mastodon.v1.Status[]
  buffer?: number
  endMessage?: boolean | string
}>()

const { formatNumber } = useHumanReadableNumber()
const virtualScroller = usePreferences('experimentalVirtualScroller')

const showOriginSite = computed(() =>
  account && account.id !== currentUser.value?.account.id && getServerName(account) !== currentServer.value,
)

function getFollowedTag(status: mastodon.v1.Status): string | null {
  const followedTagNames = followedTags.map(tag => tag.name)
  const followedStatusTags = status.tags.filter(tag => followedTagNames.includes(tag.name))
  return followedStatusTags.length > 0 ? followedStatusTags[0]?.name : null
}
</script>

<template>
  <CommonPaginator v-bind="{ paginator, stream, preprocess, buffer, endMessage }" :virtual-scroller="virtualScroller">
    <template #updater="{ number, update }">
      <button id="elk_show_new_items" py-4 border="b base" flex="~ col" p-3 w-full text-primary font-bold @click="update">
        {{ $t('timeline.show_new_items', number, { named: { v: formatNumber(number) } }) }}
      </button>
    </template>
    <template #default="{ item, older, newer, active }">
      <template v-if="virtualScroller">
        <DynamicScrollerItem :item="item" :active="active" tag="article">
          <StatusCard :followed-tag="getFollowedTag(item)" :status="item" :context="context" :older="older" :newer="newer" :account="account" />
        </DynamicScrollerItem>
      </template>
      <template v-else>
        <StatusCard :followed-tag="getFollowedTag(item)" :status="item" :context="context" :older="older" :newer="newer" :account="account" />
      </template>
    </template>
    <template v-if="context === 'account' " #done="{ items }">
      <div
        v-if="showOriginSite || items.length === 0"
        p5 text-secondary text-center flex flex-col items-center gap1
      >
        <template v-if="showOriginSite">
          <span italic>{{ $t('timeline.view_older_posts') }}</span>
          <NuxtLink
            :href="account!.url" target="_blank" external
            flex="~ gap-1" items-center text-primary
            hover="underline text-primary-active"
          >
            <div i-ri:external-link-fill />
            {{ $t('menu.open_in_original_site') }}
          </NuxtLink>
        </template>
        <span v-else-if="items.length === 0">{{ $t('timeline.no_posts') }}</span>
      </div>
    </template>
  </CommonPaginator>
</template>
