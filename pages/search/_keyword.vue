<template>
  <div class="index">
    <article-list :article="article" @loadmore="loadmoreArticle"></article-list>
  </div>
</template>

<script>
  import ArticleList from '~/components/archive/list'
  export default {
    name: 'category-article-list',
    validate ({ params }) {
      return Boolean(params.keyword)
    },
    fetch({ store, params }) {
      return store.dispatch('loadArticles', params)
    },
    head () {
      return {
        title: `${this.defaultParams.keyword} | Search` }
    },
    components: {
      ArticleList
    },
    computed: {
      article() {
        return this.$store.state.article.list
      },
      defaultParams() {
        return {
          keyword: this.$route.params.keyword
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