<template>
  <div class="index">
    <article-list :article="article" @loadmore="loadmoreArticle"></article-list>
  </div>
</template>

<script>
  import ArticleList from '~/components/archive/list'
  export default {
    name: 'category-article-list',
    components: {
      ArticleList
    },
    validate({ params, store }) {
      return Boolean(params.category_slug) && store.state.category.data.list.find(category => {
        return Object.is(category.slug, params.category_slug)
      })
    },
    fetch({ store, params }) {
      return store.dispatch('loadArticles', params)
    },
    head() {
      const slug = this.defaultParams.category_slug || ''
      const title = slug.toLowerCase().replace(/( |^)[a-z]/g, (L) => L.toUpperCase())
      return {
        title: `${title} | Category`
      }
    },
    created() {
      if (!this.currentCategory) {
        this.$router.back()
      }
    },
    computed: {
      article() {
        return this.$store.state.article.list
      },
      currentCategory() {
        return this.$store.state.category.data.list.find(category => {
          return Object.is(category.slug, this.$route.params.category_slug)
        })
      },
      defaultParams() {
        return {
          category_slug: this.$route.params.category_slug
        }
      },
      nextPageParams() {
        return Object.assign({
          page: this.article.data.pagination.currentPage + 1
        }, this.defaultParams)
      }
    },
    methods: {
      loadmoreArticle() {
        this.$store.dispatch('loadArticles', this.nextPageParams)
      }
    }
  }
</script>