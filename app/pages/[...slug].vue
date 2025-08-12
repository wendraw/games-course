<script lang="ts" setup>
const route = useRoute()
const { data: page } = await useAsyncData(route.path, () => {
  return queryCollection('content').path(route.path).first()
})
</script>

<template>
  <div v-if="page" class="mx-auto px-4 py-8 container max-w-4xl">
    <ContentRenderer
      :value="page"
      class="max-w-none prose dark:prose-invert"
    />
  </div>
  <div v-else class="mx-auto px-4 py-8 text-center container max-w-4xl">
    <div class="empty-page">
      <h1 class="text-3xl font-bold mb-4">
        Page Not Found
      </h1>
      <p class="text-gray-600 mb-6">
        Oops! The content you're looking for doesn't exist.
      </p>
      <NuxtLink to="/" class="text-blue-600 underline hover:text-blue-800">
        Go back home
      </NuxtLink>
    </div>
  </div>
</template>
