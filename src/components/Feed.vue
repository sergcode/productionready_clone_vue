<template>
  <div>
    <mcv-loading
      v-if="isLoading"
      :class="{'article-preview': true}"
      :text-message="'Loading posts...'"
    />
    <mcv-error-message v-if="error" />

    <div v-if="feed" class="article-list">
      <mcv-articles-not
        v-if="feed.articlesCount === 0"
        :text-message="'No posts are here... yet.'"
      />
      <div
        class="article-preview"
        v-for="(article, index) in feed.articles"
        :key="index"
      >
        <div class="article-meta">
          <router-link
            :to="{name: 'userProfile', params: {slug: article.author.username}}"
          >
            <img :src="article.author.image" alt="" />
          </router-link>
          <div class="info">
            <router-link
              class="author"
              :to="{
                name: 'userProfile',
                params: {slug: article.author.username}
              }"
            >
              {{ article.author.username }}
            </router-link>
            <span class="date">{{ dateRegister }}</span>
          </div>
          <div class="pull-xs-right">
            <mcv-add-to-favorites
              :is-favorited="article.favorited"
              :article-slug="article.slug"
              :favorites-count="article.favoritesCount"
            />
          </div>
        </div>
        <router-link
          :to="{name: 'article', params: {slug: article.slug}}"
          class="preview-link"
        >
          <h1>{{ article.title }}</h1>
          <p>{{ article.description }}</p>
          <span>Read more...</span>
          <mcv-tag-list :tags="article.tagList" />
        </router-link>
      </div>
      <mcv-pagination
        :total="feed.articlesCount"
        :limit="limit"
        :current-page="currentPage"
        :url="baseUrl"
      />
    </div>
  </div>
</template>

<script>
import {mapState} from 'vuex'
import {actionTypes} from '@/store/modules/feed'
import McvPagination from '@/components/Pagination'
import {limit} from '@/helpers/vars'
import {stringify, parseUrl} from 'query-string'
import McvLoading from '@/components/Loading'
import McvErrorMessage from '@/components/ErrorMessage'
import McvTagList from '@/components/TagList'
import McvAddToFavorites from '@/components/AddToFavorites'
import McvArticlesNot from '@/components/ArticlesNot'

export default {
  name: 'McvFeed',
  props: {
    apiUrl: {
      type: String,
      required: true
    }
  },
  components: {
    McvPagination,
    McvLoading,
    McvErrorMessage,
    McvTagList,
    McvAddToFavorites,
    McvArticlesNot
  },
  computed: {
    ...mapState({
      isLoading: state => state.feed.isLoading,
      feed: state => state.feed.data,
      error: state => state.feed.error
    }),
    limit() {
      return limit
    },
    baseUrl() {
      return this.$route.path
    },
    currentPage() {
      return Number(this.$route.query.page || '1')
    },
    offset() {
      return this.currentPage * limit - limit
    },
    dateRegister() {
      let date
      const articlesArray = this.feed.articles

      articlesArray.forEach(article => {
        date = new Date(article.createdAt).toLocaleString('en-Us', {
          year: 'numeric',
          month: 'long',
          day: 'numeric'
        })
      })
      return date
    }
  },
  watch: {
    currentPage() {
      console.log('currentPage change')
      this.fetchFeed()
    },
    apiUrl() {
      this.fetchFeed()
    }
  },

  mounted() {
    console.log('init feed')
    this.fetchFeed()
  },
  methods: {
    fetchFeed() {
      const parsedUrl = parseUrl(this.apiUrl)
      const stringifiedParams = stringify({
        limit,
        offset: this.offset,
        ...parsedUrl.query
      })
      const apiUrlWithParams = `${parsedUrl.url}?${stringifiedParams}`
      console.log(apiUrlWithParams)
      this.$store.dispatch(actionTypes.getFeed, {apiUrl: apiUrlWithParams})
    }
  }
}
</script>

<style scoped>
.author {
  padding-bottom: 3px;
}
</style>
