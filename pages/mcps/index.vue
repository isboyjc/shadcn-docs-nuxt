<template>
  <div class="container mx-auto px-4 py-8 md:px-8">
    <ContentHero :actions="pageData.actions">
      <template #title>
        {{ pageData.title }}
      </template>
      <template #description>
        {{ pageData.description }}
      </template>
    </ContentHero>

    <div class="flex flex-col gap-6 lg:flex-row">
      <!-- 左侧导航 -->
      <div class="lg:sticky lg:top-[80px] lg:w-64 lg:self-start">
        <!-- 搜索框 -->
        <div class="mb-6">
          <h3 class="mb-3 font-medium">
            搜索
          </h3>
          <div class="flex items-center rounded-md border bg-background px-3 py-2">
            <Icon name="lucide:search" class="mr-2 size-4 text-muted-foreground" />
            <input
              v-model="searchQuery"
              type="text"
              placeholder="搜索MCP..."
              class="w-full bg-transparent text-sm outline-none placeholder:text-muted-foreground"
            >
            <UiButton
              v-if="searchQuery"
              variant="ghost"
              size="sm"
              class="size-6 p-0"
              @click="searchQuery = ''"
            >
              <Icon name="lucide:x" class="size-3" />
            </UiButton>
          </div>
        </div>

        <!-- 分类筛选 -->
        <div>
          <h3 class="mb-3 font-medium">
            分类
          </h3>
          <div class="flex flex-col gap-2">
            <UiButton
              variant="ghost"
              size="sm"
              class="justify-start px-3"
              :class="{ 'bg-primary/10 font-medium text-primary': activeCategory === 'all' }"
              @click="setActiveCategory('all')"
            >
              <Icon name="lucide:layers" class="mr-2 size-4" />
              全部
              <UiBadge v-if="activeCategory === 'all'" variant="outline" class="ml-auto">
                {{ mcps.length }}
              </UiBadge>
            </UiButton>
            <UiButton
              v-for="category in categories"
              :key="category.id"
              variant="ghost"
              size="sm"
              class="justify-start px-3"
              :class="{ '!bg-primary/10 !font-medium !text-primary': activeCategory === category.id }"
              @click="setActiveCategory(category.id)"
            >
              <Icon :name="getCategoryIcon(category.id)" class="mr-2 size-4" :class="getCategoryIconColorClass(category.id)" />
              {{ category.name }}
              <UiBadge v-if="activeCategory === category.id || activeCategory === 'all'" variant="outline" class="ml-auto">
                {{ getCategoryCount(category.id) }}
              </UiBadge>
            </UiButton>
          </div>
        </div>

        <!-- 重置筛选按钮 -->
        <UiButton
          v-if="activeCategory !== 'all' || searchQuery"
          variant="outline"
          size="sm"
          class="mt-6 w-full"
          @click="resetFilters"
        >
          <Icon name="lucide:refresh-cw" class="mr-2 size-4" />
          重置筛选
        </UiButton>
      </div>

      <!-- 右侧内容区 -->
      <div class="flex-1">
        <!-- 筛选结果统计 -->
        <div class="mb-4 flex items-center justify-between">
          <p class="text-sm text-muted-foreground">
            共收录 <span class="font-medium text-foreground">{{ filteredMcps.length }}</span> 个MCP
            <span v-if="activeCategory !== 'all'">
              在 <span class="font-medium text-foreground">{{ getCategoryName(activeCategory) }}</span> 分类下
            </span>
            <span v-if="searchQuery">
              匹配 <span class="font-medium text-foreground">"{{ searchQuery }}"</span>
            </span>
          </p>
          <div class="flex items-center gap-2">
            <UiButton variant="ghost" size="sm" class="gap-1" @click="toggleGridSize">
              <Icon :name="isCompactGrid ? 'lucide:grid-2x2' : 'lucide:grid-3x3'" class="size-4" />
              <span class="text-xs">{{ isCompactGrid ? '标准视图' : '紧凑视图' }}</span>
            </UiButton>
          </div>
        </div>

        <!-- MCP卡片网格 -->
        <div
          class="grid gap-4" :class="[
            isCompactGrid
              ? 'grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5'
              : 'grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-2 xl:grid-cols-3 2xl:grid-cols-4',
          ]"
        >
          <UiCard
            v-for="mcp in filteredMcps"
            :key="mcp.id"
            class="group h-full overflow-hidden transition-all hover:shadow-md"
            :class="{ compact: isCompactGrid }"
          >
            <NuxtLink :to="mcp.url" target="_blank" class="block h-full">
              <UiCardHeader :class="{ 'p-3 pb-2': isCompactGrid, 'p-4 pb-2': !isCompactGrid }">
                <div class="flex items-center justify-between">
                  <div class="flex items-center gap-2">
                    <div class="flex size-8 items-center justify-center rounded-md border" :class="getCategoryBorderClass(mcp.category)">
                      <Icon
                        :name="getCategoryIcon(mcp.category)"
                        class="size-4"
                        :class="getCategoryIconColorClass(mcp.category)"
                      />
                    </div>
                    <UiCardTitle :class="{ 'text-base': isCompactGrid }">
                      {{ mcp.title }}
                    </UiCardTitle>
                  </div>
                  <Icon name="lucide:external-link" class="size-3.5 text-muted-foreground opacity-0 transition-opacity group-hover:opacity-100" />
                </div>
                <UiCardDescription
                  :class="[
                    isCompactGrid ? 'mt-2 line-clamp-1 text-xs' : 'mt-2 line-clamp-2 text-sm',
                  ]"
                >
                  {{ mcp.description }}
                </UiCardDescription>
              </UiCardHeader>
              <UiCardFooter
                class="flex items-center justify-between" :class="[
                  isCompactGrid ? 'px-3 py-2' : 'px-4 py-2',
                ]"
              >
                <div class="flex w-full items-center justify-between gap-2">
                  <span
                    class="line-clamp-1 flex-1 text-muted-foreground" :class="[
                      isCompactGrid ? 'text-xs' : 'text-sm',
                    ]"
                  >{{ mcp.provider }}</span>
                  <UiBadge
                    variant="outline"
                    class="flex-shrink-0"
                    :class="[
                      isCompactGrid ? 'px-1.5 py-0 text-[10px]' : 'py-0 text-xs',
                      getCategoryColorClass(mcp.category),
                    ]"
                  >
                    {{ getCategoryName(mcp.category) }}
                  </UiBadge>
                </div>
              </UiCardFooter>
            </NuxtLink>
          </UiCard>
        </div>

        <!-- 无结果提示 -->
        <div v-if="filteredMcps.length === 0" class="mt-10 text-center">
          <Icon name="lucide:search-x" class="mx-auto mb-4 size-12 text-muted-foreground" />
          <h3 class="mb-2 text-xl font-semibold">
            未找到匹配的MCP
          </h3>
          <p class="text-muted-foreground">
            尝试使用不同的搜索词或选择其他分类
          </p>
          <UiButton class="mt-4" variant="outline" @click="resetFilters">
            重置筛选
          </UiButton>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import ContentHero from '@/components/content/Hero.vue';

interface PageData {
  title: string;
  description: string;
  actions: {
    name: string;
    leftIcon?: string;
    rightIcon?: string;
    variant?: 'default' | 'link' | 'destructive' | 'outline' | 'secondary' | 'ghost';
    to: string;
    target?: string;
  }[];
}

const config = useConfig();

const pageData = ref<PageData>({
  title: 'MCPs',
  description: '发现和探索优质的 MCP 服务器！',
  actions: [{
    name: '了解更多',
    leftIcon: 'lucide:info',
    to: '/mcps',
    variant: 'outline',
  }],
});

// 分类数据
const categories = [
  { id: 'survival', name: '生存服务器' },
  { id: 'creative', name: '创造服务器' },
  { id: 'minigame', name: '小游戏服务器' },
  { id: 'rpg', name: 'RPG服务器' },
  { id: 'skyblock', name: '空岛服务器' },
  { id: 'modded', name: '模组服务器' },
];

// MCP数据
const mcps = [
  // 生存服务器
  {
    id: 'hypixel',
    title: 'Hypixel',
    description: '全球最大的Minecraft服务器之一，提供各种小游戏和生存模式',
    provider: 'Hypixel Inc.',
    category: 'survival',
    url: 'https://hypixel.net',
  },
  {
    id: 'cubecraft',
    title: 'CubeCraft',
    description: '提供多种小游戏和生存模式的热门Minecraft服务器',
    provider: 'CubeCraft Games',
    category: 'survival',
    url: 'https://cubecraft.net',
  },
  {
    id: 'mineplex',
    title: 'Mineplex',
    description: '大型Minecraft服务器，提供独特的小游戏和生存体验',
    provider: 'Mineplex LLC',
    category: 'survival',
    url: 'https://mineplex.com',
  },
  {
    id: 'wynncraft',
    title: 'Wynncraft',
    description: '最大的Minecraft MMORPG服务器，提供丰富的任务和冒险',
    provider: 'Wynncraft',
    category: 'rpg',
    url: 'https://wynncraft.com',
  },
  {
    id: 'manacube',
    title: 'ManaCube',
    description: '提供多种游戏模式的Minecraft服务器，包括生存和小游戏',
    provider: 'ManaCube',
    category: 'survival',
    url: 'https://manacube.com',
  },

  // 创造服务器
  {
    id: 'buildtheearth',
    title: 'Build The Earth',
    description: '致力于1:1比例重建地球的创造性Minecraft项目',
    provider: 'BTE Team',
    category: 'creative',
    url: 'https://buildtheearth.net',
  },
  {
    id: 'plotsquared',
    title: 'PlotSquared',
    description: '基于地块的创造服务器，让玩家在自己的空间中自由建造',
    provider: 'PlotSquared',
    category: 'creative',
    url: 'https://plotsquared.com',
  },
  {
    id: 'creativefun',
    title: 'CreativeFun',
    description: '专注于创造模式的服务器，提供各种建筑工具和插件',
    provider: 'CreativeFun',
    category: 'creative',
    url: 'https://creativefun.net',
  },

  // 小游戏服务器
  {
    id: 'thepit',
    title: 'The Pit',
    description: 'Hypixel上的热门PVP小游戏，玩家在竞技场中战斗',
    provider: 'Hypixel Inc.',
    category: 'minigame',
    url: 'https://hypixel.net/the-pit',
  },
  {
    id: 'bedwars',
    title: 'Bed Wars',
    description: '保护你的床并摧毁敌人的床的团队PVP游戏',
    provider: 'Various',
    category: 'minigame',
    url: 'https://minecraft-server.net/bedwars',
  },
  {
    id: 'skywars',
    title: 'Sky Wars',
    description: '在浮空岛上战斗的PVP小游戏，最后存活者获胜',
    provider: 'Various',
    category: 'minigame',
    url: 'https://minecraft-server.net/skywars',
  },

  // RPG服务器
  {
    id: 'mcrpg',
    title: 'McRPG',
    description: '提供丰富RPG体验的服务器，包括职业、技能和任务',
    provider: 'McRPG',
    category: 'rpg',
    url: 'https://mcrpg.com',
  },
  {
    id: 'lordofthecraft',
    title: 'Lord of the Craft',
    description: '最古老的Minecraft角色扮演服务器之一，提供丰富的世界观和故事',
    provider: 'LotC',
    category: 'rpg',
    url: 'https://lordofthecraft.net',
  },

  // 空岛服务器
  {
    id: 'skyblock',
    title: 'SkyBlock',
    description: '经典的空岛生存模式，从一个小岛开始发展',
    provider: 'Various',
    category: 'skyblock',
    url: 'https://minecraft-server.net/skyblock',
  },
  {
    id: 'oneblock',
    title: 'OneBlock',
    description: '从一个方块开始的空岛变种，方块会不断生成新资源',
    provider: 'Various',
    category: 'skyblock',
    url: 'https://minecraft-server.net/oneblock',
  },

  // 模组服务器
  {
    id: 'ftb',
    title: 'Feed The Beast',
    description: '提供各种模组包的服务器网络，适合喜欢科技和魔法的玩家',
    provider: 'FTB',
    category: 'modded',
    url: 'https://feed-the-beast.com',
  },
  {
    id: 'tekkit',
    title: 'Tekkit',
    description: '专注于科技模组的服务器，提供工业和自动化体验',
    provider: 'Technic',
    category: 'modded',
    url: 'https://technicpack.net',
  },
  {
    id: 'pixelmon',
    title: 'Pixelmon',
    description: '结合Minecraft和宝可梦的模组服务器，可以捕捉和训练宝可梦',
    provider: 'Pixelmon',
    category: 'modded',
    url: 'https://pixelmonmod.com',
  },
];

// 状态管理
const activeCategory = ref('all');
const searchQuery = ref('');
const isCompactGrid = ref(true);

// 根据分类ID获取分类名称
function getCategoryName(categoryId: string): string {
  const category = categories.find(c => c.id === categoryId);
  return category ? category.name : '';
}

// 获取分类图标
function getCategoryIcon(categoryId: string): string {
  const iconMap: Record<string, string> = {
    survival: 'lucide:tent',
    creative: 'lucide:palette',
    minigame: 'lucide:gamepad-2',
    rpg: 'lucide:swords',
    skyblock: 'lucide:cloud',
    modded: 'lucide:puzzle',
  };
  return iconMap[categoryId] || 'lucide:folder';
}

// 获取分类颜色
function getCategoryColor(categoryId: string): string {
  const colorMap: Record<string, string> = {
    survival: '#4ade80', // 绿色
    creative: '#f472b6', // 粉色
    minigame: '#60a5fa', // 蓝色
    rpg: '#f59e0b', // 橙色
    skyblock: '#a78bfa', // 紫色
    modded: '#ef4444', // 红色
  };
  return colorMap[categoryId] || '#94a3b8'; // 默认灰色
}

// 获取分类颜色类
function getCategoryColorClass(categoryId: string): string {
  const colorClassMap: Record<string, string> = {
    survival: 'border-green-500 text-green-600 dark:text-green-400',
    creative: 'border-pink-500 text-pink-600 dark:text-pink-400',
    minigame: 'border-blue-500 text-blue-600 dark:text-blue-400',
    rpg: 'border-amber-500 text-amber-600 dark:text-amber-400',
    skyblock: 'border-purple-500 text-purple-600 dark:text-purple-400',
    modded: 'border-red-500 text-red-600 dark:text-red-400',
  };
  return colorClassMap[categoryId] || '';
}

// 获取分类图标颜色类
function getCategoryIconColorClass(categoryId: string): string {
  const colorClassMap: Record<string, string> = {
    survival: 'text-green-500 dark:text-green-400',
    creative: 'text-pink-500 dark:text-pink-400',
    minigame: 'text-blue-500 dark:text-blue-400',
    rpg: 'text-amber-500 dark:text-amber-400',
    skyblock: 'text-purple-500 dark:text-purple-400',
    modded: 'text-red-500 dark:text-red-400',
  };
  return colorClassMap[categoryId] || '';
}

// 获取分类边框颜色类
function getCategoryBorderClass(categoryId: string): string {
  const colorClassMap: Record<string, string> = {
    survival: 'border-green-200 bg-green-50 dark:border-green-900 dark:bg-green-950',
    creative: 'border-pink-200 bg-pink-50 dark:border-pink-900 dark:bg-pink-950',
    minigame: 'border-blue-200 bg-blue-50 dark:border-blue-900 dark:bg-blue-950',
    rpg: 'border-amber-200 bg-amber-50 dark:border-amber-900 dark:bg-amber-950',
    skyblock: 'border-purple-200 bg-purple-50 dark:border-purple-900 dark:bg-purple-950',
    modded: 'border-red-200 bg-red-50 dark:border-red-900 dark:bg-red-950',
  };
  return colorClassMap[categoryId] || 'border-gray-200 bg-gray-50 dark:border-gray-800 dark:bg-gray-900';
}

// 获取分类产品数量
function getCategoryCount(categoryId: string): number {
  return mcps.filter(mcp => mcp.category === categoryId).length;
}

// 设置当前活动分类
function setActiveCategory(categoryId: string): void {
  activeCategory.value = categoryId;
}

// 切换网格大小
function toggleGridSize(): void {
  isCompactGrid.value = !isCompactGrid.value;
}

// 重置所有筛选条件
function resetFilters(): void {
  activeCategory.value = 'all';
  searchQuery.value = '';
}

// 过滤产品列表
const filteredMcps = computed(() => {
  let result = mcps;

  // 按分类筛选
  if (activeCategory.value !== 'all') {
    result = result.filter(mcp => mcp.category === activeCategory.value);
  }

  // 按搜索词筛选
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    result = result.filter(mcp =>
      mcp.title.toLowerCase().includes(query)
      || mcp.description.toLowerCase().includes(query)
      || mcp.provider.toLowerCase().includes(query),
    );
  }

  return result;
});

// 设置页面元数据
useSeoMeta({
  title: `${pageData.value.title ?? '404'} - ${config.value.site.name}`,
  ogTitle: pageData.value.title,
  description: pageData.value.description,
  ogDescription: pageData.value.description,
  ogImage: config.value.site.ogImage,
  twitterCard: 'summary_large_image',
});
</script>

<style scoped>
.compact .v-card-title {
  font-size: 0.9rem;
}

/* 卡片悬停效果 */
.group {
  transition: all 0.3s ease;
}

.group:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

/* 卡片内容区域样式 */
.ui-card-header {
  transition: padding 0.3s ease;
}

/* 标签样式 */
.ui-badge {
  transition: all 0.2s ease;
}

.ui-badge:hover {
  transform: translateY(-1px);
}

/* 分类图标样式 */
.group:hover .ui-card-title {
  color: var(--primary);
}
</style>
