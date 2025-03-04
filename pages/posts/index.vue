<template>
  <div
    class="px-4 py-6 md:px-8"
    :class="[config.main.padded && 'container']"
  >
    <ContentRenderer
      :key="page._id"
      :value="page"
      :data="appConfig.custom.data"
    />
    <div>
      <div class="group grid grid-cols-1 gap-4 sm:grid-cols-2 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 [&:not(:first-child)]:mt-5">
        <PostCard v-for="article in articles" :key="article._id" :title="article.title" :description="article.description" :img="article.image" target="_self" :to="article._path">
          <template #footer>
            <div class="flex w-full items-center justify-between gap-2">
              <div class="flex flex-col items-start justify-start">
                <UiBadge variant="outline">
                  {{ article?.category || '无分类' }}
                </UiBadge>
                <span class="mt-1 text-sm text-gray-500">{{ article.date }}</span>
              </div>

              <div class="flex items-center gap-2">
                <PostAvatarGroup v-if="article.authors.length > 1">
                  <UiAvatar v-for="(author, index) in article.authors" :key="author.username" :style="{ zIndex: article.authors.length - index }" size="sm">
                    <UiAvatarImage :src="author.avatar?.src" :alt="`@${author.username}`" />
                    <UiAvatarFallback>{{ author.username.slice(0, 2) }}</UiAvatarFallback>
                  </UiAvatar>
                </PostAvatarGroup>
                <UiAvatar v-else-if="article.authors.length === 1" :key="article.authors[0].username">
                  <UiAvatarImage :src="article.authors[0].avatar?.src" :alt="`@${article.authors[0].username}`" />
                  <UiAvatarFallback>{{ article.authors[0].username.slice(0, 2) }}</UiAvatarFallback>
                </UiAvatar>
              </div>
            </div>
          </template>
        </PostCard>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import PostAvatarGroup from '@/components/post/AvatarGroup.vue';
import PostCard from '@/components/post/Card.vue';

const { page } = useContent();
const config = useConfig();
const appConfig = useAppConfig();

const { data } = useAsyncData('content', () => {
  return queryContent('posts')
    .sort({ date: -1 })
    .find();
});

interface Post extends ParsedContent {
  description?: string;
  image?: string;
}

const articles = computed(() => {
  return data.value?.filter((v: Post) => v._path !== '/posts');
});
// console.log(articles.value)

useSeoMeta({
  title: `${page.value?.title ?? '404'} - ${config.value.site.name}`,
  ogTitle: page.value?.title,
  description: page.value?.description,
  ogDescription: page.value?.description,
  ogImage: config.value.site.ogImage,
  twitterCard: 'summary_large_image',
});
</script>
