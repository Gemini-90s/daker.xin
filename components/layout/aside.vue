<template>
  <aside class="aside">
    <div class="aside-article">
      <p class="title">
        <i class="iconfont icon-list"></i>
        <span>热门文章</span>
      </p>
      <ul class="aside-article-list" v-if="!article.fetching && article.data.list.length">
        <li class="item" :key="item.id" v-for="item in article.data.list.slice(0, 10)">
          <span class="index"></span>
          <nuxt-link class="title"
                     :title="`${item.title} - [ ${item.meta.comments} 条评论  |  ${item.meta.likes} 人喜欢 ]`"
                     :to="`/article/${item.id}`">
            <span>{{ item.title }}</span>
          </nuxt-link>
        </li>
      </ul>
    </div>
    <div class="aside-calendar">
      <calendar></calendar>
    </div>
    <div class="aside-fixed-box" :class="{ fixed: fixedMode.fixed }" v-scroll-top>
      <div class="aside-tag">
        <ul class="aside-tag-list" v-if="!tag.fetching && tag.data.list.length">
          <nuxt-link tag="li"
                     class="item"
                     :key="index"
                     :to="`/tag/${item.slug}`"
                     v-for="(item, index) in tag.data.list">
            <a class="title" :title="item.description">
              <i class="iconfont"
                 :class="[item.extends.find(t => Object.is(t.name, 'icon')).value]"
                 v-if="item.extends.find(t => Object.is(t.name, 'icon'))"></i>
              <span>&nbsp;</span>
              <span>{{ item.name }}</span>
              <span>({{ item.articleTotal || 0 }})</span>
            </a>
          </nuxt-link>
        </ul>
      </div>
      <div class="aside-tools">
        <div class="scroll-top" @click="setFullColumu">
          <span>返回顶部</span>
          <span>&nbsp;&nbsp;</span>
          <i class="iconfont icon-scroll-top"></i>
        </div>
        <div class="full-page" @click="fullScreen">
          <span>全屏阅读</span>
          <span>&nbsp;&nbsp;</span>
          <i class="iconfont icon-fullscreen"></i>
        </div>
      </div>
    </div>
  </aside>
</template>

<script>
  import { mapState } from 'vuex'
  import Calendar from './calendar.vue'
  export default {
    name: 'layout-aside',
    data() {
      return {
        fixedMode: {
          fixed: false,
          element: null,
          sidebarFixedOffsetTop: 0
        }
      }
    },
    components: {
      Calendar
    },
    computed: {
      tag() {
        return this.$store.state.tag
      },
      article() {
        return this.$store.state.article.hot
      },
      isArticlePage() {
        return this.$route.name === 'article-article_id'
      }
    },
    methods: {
      setFullColumu() {
        // this.$store.commit('option/SET_ERROR_COLUMU', true)
      },
      fullScreen() {
        this.setFullColumu()
        const docElm = document.documentElement
        // const docElm = document.getElementById('main-content')
        const requestEvent = docElm.requestFullscreen ||
          docElm.mozRequestFullScreen ||
          docElm.webkitRequestFullScreen ||
          docElm.msRequestFullscreen
        if (requestEvent) {
          requestEvent.bind(docElm)()
        }
      },
      parseScroll() {
        const element = this.fixedMode.element
        const sidebarFixedOffsetTop = this.fixedMode.sidebarFixedOffsetTop
        const windowScrollTop = document.documentElement.scrollTop ||
          window.pageYOffset ||
          window.scrollY ||
          document.body.scrollTop
        const newSidebarFixedOffsetTop = element.offsetTop
        this.fixedMode.sidebarFixedOffsetTop = newSidebarFixedOffsetTop !== sidebarFixedOffsetTop &&
          newSidebarFixedOffsetTop !== 77
          ? newSidebarFixedOffsetTop
          : sidebarFixedOffsetTop
        const isFixed = windowScrollTop > sidebarFixedOffsetTop
        this.fixedMode.fixed = isFixed && element
      }
    },
    directives: {
      scrollTop: {
        inserted(element, b, VNode) {
          // context
          const context = VNode.context
          // element
          context.fixedMode.element = element
          // 检测此元素相对于文档Document原点的绝对位置，并且这个值是不变化的
          context.fixedMode.sidebarFixedOffsetTop = element.offsetTop
          // 初始化应用
          context.parseScroll()
          // 监听滚动事件
          window.addEventListener('scroll', context.parseScroll, { passive: true })
        },
        unbind(element, b, VNode) {
          window.removeEventListener('scroll', VNode.context.parseScroll)
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  $aside-width: 19em;
  aside {
    float: right;
    display: block;
    width: $aside-width;
    margin: 0;
    .aside-search,
    .aside-article,
    .aside-calendar,
    .aside-ad,
    .aside-tag, {
      background-color: $module-bg;
    }
    .aside-search {
      margin-bottom: 1em;
      > .search-box {
        padding: .5em;
        overflow: hidden;
        > .search-input,
        > .search-btn,
        > .sitemap-btn {
          background-color: $module-hover-bg;
          height: 2em;
          line-height: 2em;
          float: left;
          &:hover {
            background-color: darken($module-hover-bg, 20);;
          }
        }
        > .search-input {
          margin-right: 0;
          width: calc(100% - 6.5em - 1px);
          box-sizing: border-box;
        }
        > .search-btn {
          width: 2em;
          background-color: darken($module-hover-bg, 20);;
          &:hover {
            background-color: darken($module-hover-bg, 40);
          }
        }
        > .sitemap-btn {
          text-align: center;
          float: right;
          width: 4em;
        }
      }
    }
    > .aside-article {
      overflow: hidden;
      margin-bottom: 1em;
      > .title {
        height: 3em;
        line-height: 3em;
        margin: 0;
        padding: 0 .8em;
        border-bottom: 1px dashed $body-bg;
        .iconfont {
          margin-right: .5em;
        }
      }
      > .aside-article-list {
        list-style: none;
        padding: .5em 0;
        margin-top: 0;
        margin-bottom: 0;
        counter-reset: hot-article-list;
        .item {
          display: block;
          height: 1.9em;
          line-height: 1.9em;
          padding: 0 .8em;
          margin-bottom: .5em;
          // color: $text-dark;
          @include text-overflow();
          &:last-child {
            margin: 0;
          }
          .index {
            color: $secondary;
            counter-increment: hot-article-list;
            background-color: $module-hover-bg;
            width: 1.5em;
            height: 1.5em;
            line-height: 1.5em;
            display: inline-block;
            text-align: center;
            margin-right: .5em;
            font-size: .8em;
            &::before {
              content: counter(hot-article-list);
            }
          }
          .title {
            font-size: .9em;
            &:hover {
              margin-left: .5em;
              text-decoration: underline;
            }
          }
        }
      }
    }
    .aside-calendar {
      padding: .8em;
      margin-bottom: 1em;
    }

    .aside-fixed-box {
      width: $aside-width;
      &.fixed {
        position: fixed;
        top: 5.5em;
        > .aside-tag {
          max-height: calc(100vh - 8em - 4.5em - 3em - 8em);
          overflow-y: auto;
        }
      }
      > .aside-tools {
        > .scroll-top {
          margin-right: 1em;
        }
        > .scroll-top,
        > .full-page {
          display: inline-block;
          width: calc((100% - 1em) / 2;
          height: 2.5em;
          line-height: 2.5em;
          text-align: center;
          background-color: $module-bg;
          cursor: pointer;
          &:hover {
            background-color: $module-hover-bg;
          }
        }
      }
      > .aside-tag {
        width: 19em;
        padding-left: 1em;
        border-top: 1em solid transparent;
        border-bottom: 1em solid transparent;
        margin-bottom: 1em;
        .empty-box {
          padding-right: .8em;
          padding-bottom: .8em;
        }
        .aside-tag-list {
          list-style: none;
          padding: 0;
          margin: 0;
          overflow: hidden;
          .item {
            display: inline-block;
            float: left;
            margin-right: 1em;
            margin-bottom: 1em;
            height: 2em;
            line-height: 2em;
            text-transform: capitalize;
            background-color: $module-hover-bg;
            &:hover {
              background-color: darken($module-hover-bg, 40);
            }
            &:last-child {
              margin: 0;
            }
            .title {
              display: block;
              padding: 0 .5em;
              font-family: $sans-serif;
            }
          }
        }
      }
    }
  }
</style>