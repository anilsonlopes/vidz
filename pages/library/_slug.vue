<template>
  <div>
    <h2 class="uppercase font-serif text-grey-light ml-2 pb-1 mt-4 border-b border-grey-dark">
      {{ librariesLabel[$route.params.slug] }}
    </h2>
    <nf-message
      v-if="loaded == -1"
      emoji="👽"
      :body="`First, add something in ${librariesLabel[$route.params.slug]}`"
      class="px-10"
    />
    <transition enter-active-class="animated fadeInUp faster">
      <div v-if="loaded == 2" class="py-2 px-1">
        <div
          v-for="(post, index) in posts"
          :key="post.id"
          class="md:mb-8"
        >
          <nuxt-link
            v-if="post"
            class="overflow-hidden flex flex-col sm:flex-row w-full no-underline hover:bg-grey-lighter p-2 transition"
            :class="{ 'pt-8 sm:pt-2': index > 0 }"
            :to="{ name: 'post-slug', params: { slug: post.slug } }"
          >
            <div>
              <div
                class="h-64 bg-grey bg-cover bg-center"
                :style="{ 'width': '13rem', 'background-image': `url(${post.poster})` }"
              />
            </div>
            <div class="md:w-full flex flex-col md:justify-between h-auto md:h-64 pt-4 sm:pl-4">
              <div>
                <div class="rounded px-4 py-1 mb-2 text-xxs text-grey-dark bg-grey-lightest shadow inline-block uppercase font-mono">
                  {{ post.type }}
                </div>
                <div class="text-grey-darker md:text-3xl">
                  <h3 class="text-grey font-thin font-serif text-xl uppercase">
                    {{ post.title }}
                  </h3>
                </div>
                <div class="sm:block max-w-sm text-grey-dark pt-2">
                  {{ post.plot | truncate(99) }}
                </div>
              </div>
              <div class="text-xs font-mono uppercase pt-2">
                <div class="pb-1 text-grey-dark">
                  {{ post.genres }}
                </div>
              </div>
            </div>
          </nuxt-link>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { db, parseData } from '~/services/firebase'
import { mapGetters } from 'vuex'

export default {
  transition: 'fadeInBottom',
  head: {
    title: 'Biblioteca'
  },
  watchQuery: true,
  layout: 'tube',
  data: () => ({
    libraries: [],
    posts: [],
    loaded: 0,
    librariesLabel: {
      'liked': 'Liked',
      'watched': 'Watched',
      'watch-later': 'Watch later'
    }
  }),
  computed: {
    ...mapGetters(['auth'])
  },
  watch: {
    auth() {
      this.fetchLibraries()
    }
  },
  mounted() {
    this.fetchLibraries()
  },
  methods: {
    fetchLibraries() {
      if (this.loaded === 0 && this.auth) {
        this.loaded = 1
        let librariesRef = db.collection('libraries')
        librariesRef = librariesRef.where('userId', '==', this.auth.uid)
        librariesRef = librariesRef.where('slug', '==', this.$route.params.slug)
        librariesRef.get().then((querySnapshot) => {
          this.loaded = querySnapshot.empty ? -1 : this.loaded
          this.libraries = parseData(querySnapshot)
          this.fetchPosts()
        })
      }
    },
    fetchPosts() {
      this.libraries.map((library) => {
        const ref = db.collection('posts').where('id', '==', library.postId)
        ref.get().then((querySnapshot) => {
          this.posts.push(parseData(querySnapshot)[0])
          this.loaded = 2
        })
      })
    }
  }
}
</script>
