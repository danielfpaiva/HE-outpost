<template>
  <div class="homepage">
    <template v-if="loading">
      <loading />
    </template>

    <template v-else>
      <h2 class="section-title mb-4">Featured Posts</h2>

      <div class="post-highlights">
        <post-summary v-for="(post,i) of curated" :key="i" :post="post" type="feed" />
      </div>

      <div v-if="!trendingIsCurated" class="mt-4 text-right">
        <nuxt-link :to="{name:'sort', params:{sort:'curated'}}" class="btn btn-outline-primary btn-sm">
          See more curated content <fa-icon icon="angle-right" />
        </nuxt-link>
      </div>

      <div class="row mt-5">
        <div class="col-lg-6 mb-4">
          <h2 class="section-title mb-4">Popular</h2>
          <post-summary v-for="(post,i) of trending" :key="i" :post="post" type="feed" />

          <div class="mt-4 text-right">
            <nuxt-link :to="{name:'sort', params:{sort:'trending'}}" class="btn btn-outline-primary btn-sm">
              See more popular content <fa-icon icon="angle-right" />
            </nuxt-link>
          </div>
        </div>

        <div class="col-lg-6 mb-4">
          <h2 class="section-title mb-4">Latest</h2>
          <post-summary v-for="(post,i) of created" :key="i" :post="post" type="feed" />

          <div class="mt-4 text-right">
            <nuxt-link :to="{name:'sort', params:{sort:'created'}}" class="btn btn-outline-primary btn-sm">
              See more new content <fa-icon icon="angle-right" />
            </nuxt-link>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import postIndex from '@/mixins/postIndex'

export default {
  name: 'Home',

  mixins: [postIndex],

  data () {
    return {
      curated: [],
      trending: [],
      created: [],

      trendingIsCurated: false
    }
  },

  async fetch () {
    this.loading = true

    const params = this.$config.CURATED_FEED ? {} : { limit: 15 }

    const requests = [
      this.fetchPosts({ endpoint: 'get_discussions_by_trending', params }),
      this.fetchPosts({ endpoint: 'get_discussions_by_created' })
    ]

    if (this.$config.CURATED_FEED) {
      requests.push(this.fetchPosts({ endpoint: 'curated' }))
    }

    // eslint-disable-next-line prefer-const
    let [trending, created, curated] = await Promise.all(requests)

    if (!curated || curated.length <= 0) {
      this.trendingIsCurated = true

      curated = trending.splice(0, 5)
    } else {
      curated = curated.splice(0, 5)
    }

    this.curated = curated
    this.trending = trending
    this.created = created

    this.loading = false
  }
}
</script>

<style lang="scss">
.section-title {
  font-size: 1.5rem;
  font-weight: 600;
  position: relative;
  padding-bottom: 0.5rem;

  &:after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    width: 50px;
    height: 3px;
    background-color: #0d6efd;
    border-radius: 3px;
  }
}
</style>
