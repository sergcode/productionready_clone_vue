<template>
  <div class="profile-page">
    <div class="user-info">
      <div class="container">
        <div class="row">
          <mcv-loading
            v-if="isLoading"
            :class="{
              loading__center: true,
              'loading__font-size': true,
              'pb-1': true
            }"
            :text-message="'Loading avatar and username user...'"
            :spinner="true"
          />
          <mcv-error-message v-if="error" />
          <div v-if="userProfile" class="col-xs-12 col-md-10 offset-md-1">
            <img alt="" class="user-img" :src="userProfile.image" />
            <h4>{{ userProfile.username }}</h4>
            <p>{{ userProfile.bio }}</p>
            <div>
              <mcv-follow-user
                v-if="!isCurrentUserProfile"
                :is-followed="userProfile.following"
              />

              <router-link
                v-if="isCurrentUserProfile"
                class="btn btn-sm btn-outline-secondary action-btn"
                :to="{name: 'settings'}"
              >
                <i class="ion-gear-a"></i>
                Edit Profile Settings
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="userProfile" class="container">
      <div class="row">
        <div class="col-xs-12 col-md-10 offset-md-1">
          <div class="articles-toggle">
            <ul class="nav nav-pills outline-active">
              <li class="nav-item">
                <router-link
                  :to="{
                    name: 'userProfile',
                    params: {slug: userProfile.username}
                  }"
                  class="nav-link"
                  :class="{active: routeName === 'userProfile'}"
                >
                  My Post
                </router-link>
              </li>

              <li class="nav-item">
                <router-link
                  :to="{
                    name: 'userProfileFavorites',
                    params: {slug: userProfile.username}
                  }"
                  class="nav-link"
                  :class="{active: routeName === 'userProfileFavorites'}"
                >
                  Favorites Posts
                </router-link>
              </li>
            </ul>
          </div>
          <mcv-feed :api-url="apiUrl" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {mapState, mapGetters} from 'vuex'
import {actionTypes as userProfileActionTypes} from '@/store/modules/userProfile'
import {getterTypes as authGetterTypes} from '@/store/modules/auth'
import McvLoading from '@/components/Loading'
import McvErrorMessage from '@/components/ErrorMessage'
import McvFeed from '@/components/Feed'
import McvFollowUser from '@/components/FollowUser'

export default {
  name: 'McvUserProfile',
  components: {
    McvLoading,
    McvErrorMessage,
    McvFeed,
    McvFollowUser
  },
  computed: {
    ...mapState({
      isLoading: state => state.userProfile.isLoading,
      error: state => state.userProfile.error,
      userProfile: state => state.userProfile.data
    }),
    ...mapGetters({
      currentUser: authGetterTypes.currentUser
    }),
    isCurrentUserProfile() {
      if (!this.currentUser || !this.userProfile) {
        return false
      }
      return this.currentUser.username === this.userProfile.username
    },
    userProfileSlug() {
      return this.$route.params.slug
    },
    apiUrl() {
      const isFavorites = this.$route.path.includes('favorites')
      return isFavorites
        ? `/articles?favorited=${this.userProfileSlug}`
        : `/articles?author=${this.userProfileSlug}`
    },
    routeName() {
      return this.$route.name
    }
  },
  watch: {
    userProfileSlug() {
      this.getUserProfile()
    }
  },
  mounted() {
    this.getUserProfile()
  },
  methods: {
    getUserProfile() {
      this.$store.dispatch(userProfileActionTypes.getUserProfile, {
        slug: this.userProfileSlug
      })
    }
  }
}
</script>
