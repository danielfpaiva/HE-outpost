<template>
  <div class="post-card">
    <div v-if="thumbnail" class="post-card-image-container">
      <div v-if="!shouldShowPost" class="nsfw-overlay">
        <b-badge variant="danger">NSFW</b-badge>
        <button class="btn btn-sm btn-light mt-2" @click="toggle">Reveal</button>
      </div>

      <nuxt-link v-else :to="{ name:'user-post', params: { user: post.author, post: post.permlink }}">
        <img :src="largeThumbnail" class="post-card-image" :alt="post.title">
      </nuxt-link>
    </div>

    <div class="post-card-content">
      <div v-if="type === 'user-feed' && post.author !== user" class="reblog-text mb-2">
        <fa-icon icon="redo" /> reblogged
      </div>

      <div v-else-if="type === 'feed' && rebloggedBy.length > 0" class="reblog-text mb-2">
        <fa-icon icon="redo" /> 
        <nuxt-link v-for="(reblogger, i) of rebloggedBy" :key="i" :to="{name:'user', params:{user: reblogger}}">
          {{ reblogger }}
        </nuxt-link> reblogged
      </div>

      <div class="post-card-meta">
        <div class="post-card-author">
          <nuxt-link :to="{name:'user', params:{user:post.author}}">
            <img :src="`${$config.IMAGES_CDN}u/${post.author}/avatar`" class="post-card-author-avatar" alt="Author">
          </nuxt-link>

          <div>
            <nuxt-link class="font-weight-bold" :to="{name:'user', params:{user:post.author}}">
              @{{ post.author }}
            </nuxt-link>

            <b-badge variant="info" class="ml-1">
              {{ getReputation(post.author) }}
            </b-badge>

            <div class="small text-muted">
              <timeago :datetime="createdAt" :title="createdAt.toLocaleString()" :auto-update="60" />

              <template v-if="type !== 'comments'">
                <span class="mx-1">•</span>
                <template v-if="post.parent_permlink">
                  <nuxt-link :to="{name:'sort-tag', params:{sort:'trending', tag: post.parent_permlink}}">
                    {{ getCommunity(post.parent_permlink) }}
                  </nuxt-link>
                </template>
                <template v-else>
                  {{ getCommunity(post.parent_permlink) }}
                </template>
              </template>
            </div>
          </div>
        </div>

        <div v-if="type !== 'comments' && post.score_promoted > 0" class="ml-auto">
          <b-badge variant="warning" class="text-uppercase">Promoted</b-badge>
        </div>
      </div>

      <template v-if="!shouldShowPost">
        <template v-if="$auth.loggedIn">
          <a class="cursor-pointer" @click.prevent="showNsfw = true">Reveal this post</a> or adjust your 
          <nuxt-link :to="{name:'user-settings', params:{user:$auth.user.username}}">display preferences</nuxt-link>
        </template>
        <template v-else>
          <a class="cursor-pointer" @click.prevent="showNsfw = true">Reveal this post</a>
        </template>
      </template>

      <template v-else>
        <nuxt-link :to="{ name:'user-post', params: { user: post.author, post: post.permlink }}" class="post-card-title">
          {{ post.title }}
        </nuxt-link>

        <nuxt-link class="post-card-excerpt" :to="{ name:'user-post', params: { user: post.author, post: post.permlink }}">
          {{ extractBodySummary(post.desc) }}
        </nuxt-link>
      </template>

      <div class="post-card-stats mt-3">
        <div class="post-card-stat">
          <votes
            :author="post.author"
            :permlink="post.permlink"
            :active-votes="post.active_votes"
            :rshares="post.vote_rshares"
            :payout="post.pending_token || post.total_payout_value"
            :is-comment="false"
          />
        </div>

        <div class="post-card-stat">
          <fa-icon :icon="['far', 'comments']" /> {{ post.children }}
        </div>

        <div class="post-card-stat">
          <payout :post="post" />
        </div>

        <div class="post-card-stat">
          <extra-actions :post="post" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { proxifyImageUrl } from '@/utils/proxify-url'
import { extractImageLink, extractBodySummary } from '@/utils/extract-content'
import Votes from '@/components/Votes.vue'
import ExtraActions from '@/components/ExtraActions.vue'
import Payout from '@/components/Payout.vue'

export default {
  name: 'PostSummary',

  components: {
    Votes,
    Payout,
    ExtraActions
  },

  props: {
    post: { type: Object, required: true },
    user: { type: String, default: 'null' },
    type: { type: String, default: 'feed' }
  },

  data () {
    return {
      showNsfw: false,
      nsfwPref: 'warn'
    }
  },

  computed: {
    ...mapGetters(['tribe_info']),
    ...mapGetters('scot', ['communities', 'accounts']),

    thumbnail () {
      return extractImageLink(this.post.json_metadata, this.post.desc)
    },

    largeThumbnail () {
      if (!this.thumbnail) { return }

      return proxifyImageUrl(this.thumbnail, '640x480').replace(/ /g, '%20')
    },

    mediumThumbnail () {
      if (!this.thumbnail) { return }

      return proxifyImageUrl(this.thumbnail, '256x512').replace(/ /g, '%20')
    },

    createdAt () {
      return new Date(`${this.post.created}Z`)
    },

    cashoutTime () {
      return new Date(`${this.post.cashout_time}Z`)
    },

    rebloggedBy () {
      if (!this.post.reblogged_by) {
        return []
      }

      return this.post.reblogged_by.filter(r => r !== this.post.author)
    },

    shouldShowPost () {
      if (this.post.is_nsfw && this.nsfwPref === 'warn') {
        return this.showNsfw
      }

      return true
    }
  },

  created () {
    this.nsfwPref = this.$cookies.get('nsfw_pref') || 'warn'
  },

  methods: {
    extractBodySummary,

    getCommunity (tag) {
      let parentTag = tag

      if (!parentTag) {
        try {
          parentTag = JSON.parse(this.post.json_metadata).tags[0]
        } catch {

        }
      }

      const community = this.communities[parentTag]

      return community ? community.title : parentTag
    },

    getReputation (author) {
      const account = this.accounts[author]

      return account ? account.reputation : ''
    },

    toggle () {
      this.showNsfw = !this.showNsfw
    }
  }
}
</script>

<style lang="scss">
.post-card {
  background-color: #ffffff;
  border-radius: 0.75rem;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.03);
  margin-bottom: 1.5rem;
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease;

  &:hover {
    transform: translateY(-3px);
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.08);
  }
}

.post-card-image-container {
  position: relative;
  width: 100%;
  height: 220px;
  overflow: hidden;
}

.post-card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;

  &:hover {
    transform: scale(1.05);
  }
}

.nsfw-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
}

.post-card-content {
  padding: 1.25rem;
}

.reblog-text {
  color: #0d6efd;
  font-style: italic;
  font-size: 0.875rem;
}

.post-card-meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1rem;
}

.post-card-author {
  display: flex;
  align-items: center;
}

.post-card-author-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin-right: 0.75rem;
  border: 2px solid #f8f9fa;
}

.post-card-title {
  display: block;
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  color: #212529;
  line-height: 1.4;

  &:hover {
    color: #0d6efd;
    text-decoration: none;
  }
}

.post-card-excerpt {
  display: block;
  color: #6c757d;
  margin-bottom: 1rem;

  &:hover {
    color: #495057;
    text-decoration: none;
  }
}

.post-card-stats {
  display: flex;
  justify-content: space-between;
  color: #6c757d;
  font-size: 0.875rem;
  border-top: 1px solid #f0f2f5;
  padding-top: 1rem;
}

.post-card-stat {
  display: flex;
  align-items: center;

  .svg-inline--fa {
    margin-right: 0.25rem;
  }
}
</style>