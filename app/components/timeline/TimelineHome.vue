<script setup lang="ts">
import type { mastodon } from 'masto'

const { isSupported, effectiveType } = useNetwork()
const isSlow = computed(() => isSupported.value && effectiveType.value && ['slow-2g', '2g', '3g'].includes(effectiveType.value))
const limit = computed(() => isSlow.value ? 10 : 30)

const paginator = useMastoClient().v1.timelines.home.list({ limit: limit.value })
const stream = useStreaming(client => client.user.subscribe())
function reorderAndFilter(items: mastodon.v1.Status[]) {
  return reorderedTimeline(items, 'home')
}

let followedTags: mastodon.v1.Tag[] | undefined
if (currentUser.value !== undefined) {
  followedTags = (await useMasto().client.value.v1.followedTags.list({ limit: 0 }))
}
</script>

<template>
  <div>
    <PublishWidgetList draft-key="home" />
    <div h="1px" w-auto bg-border mb-3 />
    <TimelinePaginator :followed-tags="followedTags" v-bind="{ paginator, stream }" :preprocess="reorderAndFilter" context="home" />
  </div>
</template>
