<script setup lang="ts">
import { getProcessor } from 'bytemd'
import { visit } from 'unist-util-visit'
import type { ICatalogue } from '@/types/IArticleItem'
import type { IArticle } from '@/types/IArticle'
const props = defineProps<{
  article: IArticle
}>()
const catalogueList = ref<ICatalogue[]>([]) // 目录
const stringifyHeading = (e: any) => {
  let result = ''
  visit(e, (node) => {
    if (node.type === 'text')
      result += node.value
  })
  return result
}
getProcessor({
  plugins: [
    {
      rehype: p =>
        p.use(() => (tree: any) => {
          if (tree && tree.children.length) {
            const items: { level: number; text: string }[] = []
            tree.children
              .filter((v: any) => v.type === 'element')
              .forEach((node: any) => {
                // 过滤掉主题和高亮
                const removeTheme = node.children.filter((item: any) => item.value?.includes('theme'))
                const removeHl = node.children.filter((item: any) => item.value?.includes('highlight'))

                if (node.tagName[0] === 'h' && !!node.children.length && removeTheme.length === 0 && removeHl.length === 0) {
                  const i = Number(node.tagName[1])
                  items.push({
                    level: i,
                    text: stringifyHeading(node),
                  })
                }
              })
            catalogueList.value = items.filter(v => v.level === 1 || v.level === 2 || v.level === 3)
          }
        }),
    },
  ],
}).processSync(props.article.content)

const { immerseState } = useImmerse()
</script>

<template>
  <div class="sidebar">
    <ArticlesContentSideBarRightAuthor v-if="!immerseState" :author="article!.authorId" :viewed="article.viewed" :liked="article.liked" />
    <ArticlesContentSideBarRightRelatedArticles v-if="!immerseState" class="sidebar-block" :author="article!.authorId" :tags="article!.tagIds" />
    <div class="sticky-block-box">
      <ArticlesContentSideBarRightCatalogue v-if="catalogueList.length !== 0" class="sidebar-block" :catalogue-list="catalogueList" />
      <ArticlesContentSideBarRightColumn v-show="!immerseState" v-if="article?.columId.data[0]" :column="article?.columId.data[0]" />
    </div>
  </div>
</template>

<style scoped>
.sidebar-block {
  position: relative;
  margin-bottom: 1.5rem;
}
.sidebar .sidebar-block {
  margin-bottom: 20px;
}
.sidebar {
  position: absolute;
  top: 0;
  right: 0;
  width: 25rem;
}
.sidebar.sticky .sticky-block-box {
  position: fixed;
  top: 6.766999999999999rem;
  width: inherit;
  transition: top 0.2s;
}
.sidebar.sticky.top .sticky-block-box {
  top: 1.767rem;
}
@media (max-width: 1000px) {
  .sidebar {
    display: none;
  }
}
</style>
