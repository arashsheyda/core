<script setup lang="ts">
import { joinURL } from 'ufo'

const { data: page } = await useAsyncData('templates', () => queryContent('/templates').findOne())

const { url } = useSiteConfig()
// const { templates, fetchList } = useTemplate()

const templates = ref([{"id":1,"slug":"starter","owner":"nuxt-hub","repo":"starter","branch":"main","title":"NuxtHub Starter","description":"A minimal starter to get started with NuxtHub storage and features.","demoUrl":null,"imageUrl":"https://assets.hub.nuxt.com/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cmwiOiJodHRwczovL2h1Yi1zdGFydGVyLm51eHQuZGV2IiwiaWF0IjoxNzEyMDQ5NjEyfQ.I8ooZK96hYTigKTaExQspIP7QKHtP9kCHATy_6TnaNs.jpg","published":true,"workersPaid":false,"features":["database","kv","blob","cache"],"createdAt":"2024-03-28T21:12:54.000Z"},{"id":2,"slug":"todos","owner":"atinux","repo":"nuxt-todos-edge","branch":"main","title":"Nuxt Todos","description":"A full-stack application with authentication and a database to manage your Todos.","demoUrl":"https://todos.nuxt.dev","imageUrl":"https://assets.hub.nuxt.com/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cmwiOiJodHRwczovL251eHQtdG9kb3MtZWRnZS5wYWdlcy5kZXYiLCJpYXQiOjE3MTEzNjM3NjV9.T3QNfNtEiyQv6vsrwfbYx1yFlPbDq4PO41RtXXUEdMc.jpg","published":true,"workersPaid":false,"features":["database"],"createdAt":"2024-03-28T21:12:54.000Z"},{"id":3,"slug":"image-gallery","owner":"flosciante","repo":"nuxt-image-gallery","branch":"main","title":"Nuxt Image Gallery","description":"An image gallery to upload, edit and share your images to the world.","demoUrl":"https://image-gallery.nuxt.dev/","imageUrl":"https://assets.hub.nuxt.com/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cmwiOiJodHRwczovL251eHQtaW1hZ2UtZ2FsbGVyeS5wYWdlcy5kZXYiLCJpYXQiOjE3MTA4NzAyMjd9.n97Cf7vLFd085tbV4Gqws0xRfNqVKKBI6NV3mfl7soE.jpg","published":true,"workersPaid":false,"features":["blob"],"createdAt":"2024-03-28T21:12:54.000Z"},{"id":4,"slug":"atinotes","owner":"atinux","repo":"atinotes","branch":"main","title":"Atinotes","description":"An editable website powered by Markdown & Vue components with dynamic OG image generation.","demoUrl":"https://notes.atinux.com","imageUrl":"https://assets.hub.nuxt.com/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cmwiOiJodHRwczovL25vdGVzLmF0aW51eC5jb20iLCJpYXQiOjE3MTA4NzM5MDN9.4OB91tp5bs_s_ZQ-6ybo_CkNmzE0XOTNKR_wzvkeO_E.jpg","published":true,"workersPaid":true,"features":["kv"],"createdAt":"2024-03-28T21:12:54.000Z"}])

useSeoMeta({
  title: page.value.title,
  ogTitle: page.value.title,
  description: page.value.description,
  ogDescription: page.value.description,
  ogImage: joinURL(url, '/social-card.png')
})

// await fetchList()
</script>

<template>
  <UContainer>
    <UPageHeader align="center" class="my-16" v-bind="page.hero" :ui="{ title: 'w-full text-center', wrapper: 'border-0' }" />
    <UPageGrid>
      <UPageCard v-for="(template, index) in templates" :key="index"
        :ui="{ wrapper: 'overflow-hidden flex flex-col', body: { base: '!p-0 relative group' } }">
        <div class="w-full aspect-[16/9] h-full">
          <div v-if="template.demoUrl">
            <img :src="template.imageUrl"
              class="object-cover object-top w-full h-full xl:hidden" :alt="template.title" width="600" height="300"
              format="webp" :modifiers="{ pos: 'top' }" />
            <img :src="template.imageUrl"
              class="object-cover object-top w-full h-full hidden xl:block" :alt="template.title" width="280"
              height="140" format="webp" :modifiers="{ pos: 'top' }" />
          </div>

          <div v-else
            class="w-full h-full bg-gray-200 dark:bg-gray-800 flex flex-col gap-2 items-center justify-center text-gray-400 dark:text-gray-500 font-semibold">
            <UIcon name="i-ph-camera-slash-duotone" class="w-8 h-8" />
            <span>No preview</span>
          </div>
        </div>

        <div class="group-hover:opacity-100 opacity-0 absolute inset-0 bg-gray-900/40 transition-opacity">
          <div class="h-full flex flex-col items-center justify-center gap-2">
            <UButton label="View on GitHub" color="black" size="lg" icon="i-simple-icons-github" trailing-icon="i-ph-arrow-up-right" :to="`https://github.com/${template.repo}`" external />
            <UButton v-if="template.demoUrl" label="View demo" size="lg" :to="template.demoUrl"
              color="gray" trailing-icon="i-ph-arrow-up-right" :ui="{ color: { gray: { solid: 'hover:dark:bg-gray-700' } } }" external />
          </div>
        </div>

        <template #footer>
          <div class="flex flex-col items-start gap-2">
            <h3 class="inline-flex items-center gap-2 font-medium">
              {{ template.title }}
            </h3>
            <p class="text-sm">
              {{ template.description }}
            </p>
          </div>
        </template>
      </UPageCard>
    </UPageGrid>
  </UContainer>
</template>
