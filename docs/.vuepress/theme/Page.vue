<template>
  <div class="article">
    <div class="article-wrp">
      <TransitionContent :aniType="aniType">
        <Content :custom="false"/>
      </TransitionContent>
      <div class="content edit-link" v-if="editLink">
        <a :href="editLink" target="_blank" rel="noopener noreferrer">{{ editLinkText }}</a>
        <OutboundLink/>
      </div>
      <div class="content page-nav" v-if="prev || next">
        <p class="inner">
          <span v-if="prev" class="prev" @click="changePage('prev')">
            ← <router-link v-if="prev" class="prev" 
            :to="prev.path">
              {{ prev.title || prev.path }}
            </router-link>
          </span>
          <span v-if="next" class="next"  @click="changePage('next')">
            <router-link v-if="next" :to="next.path">
              {{ next.title || next.path }}
            </router-link> →
          </span>
        </p>
      </div>
      <slot name="bottom"/>
    </div>
  </div>
</template>

<script>
import OutboundLink from './OutboundLink.vue'
import TransitionContent from './TransitionContent.vue'
import { resolvePage, normalize, outboundRE, endingSlashRE } from './util'
//点击复制功能
import("clipboard").then(cb => {
      new cb.default(".colorBox");
    })

export default {
  components: { OutboundLink, TransitionContent },
  props: ['sidebarItems'],
  data() {
    return {
      aniType: 'no'
    }
  },
  methods: {
    changePage (changeType) {
      if (changeType === 'prev') {
        this.aniType = 'left'
        setTimeout(() => this.aniType = 'no', 1200)
      } else if (changeType === 'next') {
        this.aniType = 'right'
        setTimeout(() => this.aniType = 'no', 1200)
      }
    }
  },
  computed: {
    prev () {
      const prev = this.$page.frontmatter.prev
      if (prev === false) {
        return
      } else if (prev) {
        return resolvePage(this.$site.pages, prev, this.$route.path)
      } else {
        return resolvePrev(this.$page, this.sidebarItems)
      }
    },
    next () {
      const next = this.$page.frontmatter.next
      if (next === false) {
        return
      } else if (next) {
        return resolvePage(this.$site.pages, next, this.$route.path)
      } else {
        return resolveNext(this.$page, this.sidebarItems)
      }
    },
    editLink () {
      const {
        repo,
        editLinks,
        docsDir = '',
        docsBranch = 'master',
        docsRepo = repo
      } = this.$site.themeConfig

      let path = normalize(this.$page.path)
      if (endingSlashRE.test(path)) {
        path += 'README.md'
      } else {
        path += '.md'
      }

      if (docsRepo && editLinks) {
        const base = outboundRE.test(docsRepo)
          ? docsRepo
          : `https://github.com/${docsRepo}`
        return (
          base.replace(endingSlashRE, '') +
          `/edit/${docsBranch}/` +
          docsDir.replace(endingSlashRE, '') +
          path
        )
      }
    },
    editLinkText () {
      return (
        this.$themeLocaleConfig.editLinkText ||
        this.$site.themeConfig.editLinkText ||
        `Edit this page`
      )
    }
  }
}

function resolvePrev (page, items) {
  return find(page, items, -1)
}

function resolveNext (page, items) {
  return find(page, items, 1)
}

function find (page, items, offset) {
  const res = []
  items.forEach(item => {
    if (item.type === 'group') {
      res.push(...item.children || [])
    } else {
      res.push(item)
    }
  })
  for (let i = 0; i < res.length; i++) {
    const cur = res[i]
    if (cur.type === 'page' && cur.path === page.path) {
      return res[i + offset]
    }
  }
}
</script>

<style lang="stylus">
@import './styles/config.styl'
@import './styles/article.styl'

.edit-link.content
  padding-top 0 !important
  a
    color lighten($textDark, 25%)
    margin-right 0.25rem

.v1 .page-nav.content:not(.custom)
  padding-top 80px
  transition 1s all ease 
  .inner
    display block
    overflow hidden
    margin-top 0
    margin-bottom 0
    border-top 1px solid $borderColor
    padding-top 50px
  .next
    float right
    line-height 26px

@media (max-width: $MQMobileNarrow)
  .page-nav.content:not(.custom)
    padding-top 40px
    .inner
      padding-top 20px
</style>
