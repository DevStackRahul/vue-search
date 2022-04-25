<template>
  <div class="container">
    <div class="row">
      <div class="p-subpage__main resource-hub resource-sub">
        <div class="col-lg-8 resource-page-content">
          <p class="breadcrumb">
            <router-link v-if="this.collection.length > 0" :to="`/?` + this.getRoute()" class="back">
              <i class="icofont-long-arrow-left"></i>
            </router-link>
            <a v-if="this.collection.length < 1" :href="$router.resolve({name: 'Home'}).href" class="back">
              <i class="icofont-long-arrow-left"></i>
            </a>
            <router-link v-if="this.collection.length > 0" to="/" class="segment">Resources</router-link>
            <a v-if="this.collection.length < 1" :href="$router.resolve({name: 'Home'}).href" class="segment">Resources</a>
            <span class="divider">/</span>
            <span class="on">{{post[0].post_tag[0].name}}</span>
          </p>
          <div v-if="!loaded" class="loading">
            <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
          </div>
          <h1 class="page-title">{{showTitle}}</h1>
          <p class="page-tags">
            <span v-if="post[0].post_tag">
              <router-link v-for="t in post[0].post_tag" :key="t.slug" :to="{ name: 'Home', query: { s: this.$route.query.s, f: t.slug, t: this.$route.query.t } }">
                #{{t.name.toLowerCase()}}
              </router-link>
            </span>
            <span v-if="post[0].category">
              <router-link v-for="t in post[0].category" :key="t.slug" :to="{ name: 'Home', query: { s: this.$route.query.s, f: t.slug, t: this.$route.query.t } }">
                #{{t.name.toLowerCase()}}
              </router-link>
            </span>
            <span v-else>&nbsp;</span>
          </p>
          <div v-if="post[0].post_tag[0].name === 'Videos'" class="video-wrap">
            <div class="inner-content" v-html="post[0].post_content"></div>
            <div class="inner-description" v-html="post[0].description"></div>
          </div>
          <div v-else-if="post[0].post_tag[0].name === 'Simulations'" class="sim-wrap">
            <div class="inner-content" v-html="post[0].post_content"></div>
            <div class="inner-description" v-html="post[0].description"></div>
          </div>
          <div v-else-if="post[0].post_tag[0].name === 'Training'" class="sim-wrap training-wrap">
            <div class="inner-content" v-html="post[0].post_content"></div>
            <div class="inner-description" v-html="post[0].description"></div>
          </div>
          <div v-else class="content-wrap">
            <div class="image" v-bind:class="[post[0].post_type === 'resource' ? 'resource-type' : 'post-type']">
              <img v-if="post[0].thumbnail" class="img-thumb"
                :src="post[0].thumbnail.url"
                :alt="post[0].post_title"
              />
              <img v-else-if="post[0].featured_image" class="img-featured"
                :src="post[0].featured_image"
                :alt="post[0].post_title"
              />
              <div v-if="post[0].file" class="resource-thumb">
                  <a class="download-link" :href="post[0].file.url" target="_blank">
                    <i v-if="post[0].file.subtype == 'pdf'" class="icofont-file-pdf" />
                    <i v-if="post[0].file.subtype == 'jpeg'" class="icofont-file-jpg" />
                    <i v-if="post[0].file.subtype == 'doc'" class="icofont-file-word" />
                    <i v-if="post[0].file.subtype == 'ppt'" class="icofont-file-powerpoint" />
                    <span class="filename">{{post[0].file.filename}}</span>
                    <span class="filesize"> (<b>{{post[0].file.subtype}}</b> {{this.fileSize(post[0].file.filesize)}})</span>
                  </a>
              </div>
            </div>
            <div class="content">
              <div class="inner-content" v-html="post[0].post_content">
              </div>
              <p>
                <router-link v-if="this.collection.length > 0" to="/" class="back">
                  <i class="icofont-long-arrow-left"></i>
                </router-link>
                <a v-if="this.collection.length < 1" :href="$router.resolve({name: 'Home'}).href" class="back">
                  <i class="icofont-long-arrow-left"></i>
                </a>
              </p>
            </div>
          </div>
        </div>
        <aside class="col-lg-4 resource-aside-content">
          <h2>Related resources...</h2>
          <div class="results">
            <div class="cards">
              <div class="card" :class="'card-' + (i + 1)" v-for="(card, i) in relatedCards" :key="card.ID">
                <div class="card-content">
                 <div class="thumb">
                   <img v-if="card.thumbnail" class="img-thumb"
                      :src="card.thumbnail.url"
                      :alt="card.post_title"
                    />
                    <img v-else-if="card.featured_image" class="img-featured"
                      :src="card.featured_image"
                      :alt="card.post_title"
                    />
                    <div v-if="card.file" class="resource-thumb">
                      <i class="icofont-file-pdf" />
                      <img src="https://awrproject.com.au/wp-content/uploads/logo-1.png" alt="AWR">
                    </div>
                  </div>
                  <p class="card-type">{{card.post_type}}</p>
                  <h3>
                    <router-link :click="getPostFromThisCollection()" :to="{ name: 'Resource', params: { id: card.ID }, query: { s: this.$route.query.s, f: this.$route.query.f, t: this.$route.query.t } }">{{card.post_title}}</router-link>
                  </h3>
                  <p class="desc">
                    <!-- <span v-for="x in tagFilters" :key="x">
                      {{x}}
                    </span> -->
                  </p>
                  <p class="tags" v-if="card.category || card.post_tags" >
                    <router-link v-for="t in post[0].post_tag" :key="t.slug" :to="{ name: 'Home', query: { s: this.$route.query.s, f: t.slug, t: this.$route.query.t } }">
                      <span>#{{t.name.toLowerCase()}}</span>
                    </router-link>
                    <router-link v-for="t in post[0].category" :key="t.slug" :to="{ name: 'Home', query: { s: this.$route.query.s, f: t.slug, t: this.$route.query.t } }">
                      <span>#{{t.name.toLowerCase()}}</span>
                    </router-link>
                  </p>
                </div>
              </div>
            </div>
          </div>
        </aside>
      </div>
    </div>
  </div>
</template>

<script>
// import router from '../router'
import axios from 'axios'

export default {
  props: ['collection', 'tagFilters', 'searchValue', 'setTag'],
  data () {
    return {
      post: [],
      tmpPost: [{ post_title: '', post_excerpt: '', post_content: '', ID: 9999, permalink: 'https://awrproject.com.au/resources/' }],
      reference: 0,
      id: 0,
      loaded: false,
      loadedRelated: false,
      searchTerm: '',
      filters: [],
      relatedCollection: [],
      toggleCards: false,
      tmpCollection: []
    }
  },
  computed: {
    showTitle: function () {
      var title = this.post[0].post_title
      return title
    },
    relatedCards () {
      let relCards = this.collection

      // Filter out tag
      if (this.filters.length > 0 && this.filters) {
        relCards = relCards.filter((item) => {
          if (this.id !== item.ID) {
            if (item.post_tag !== undefined) {
              for (const prop in item.post_tag) {
                for (const val in this.filters) {
                  if (item.post_tag[prop].slug.includes(this.filters[val])) {
                    return item
                  }
                }
              }
            }
            if (item.category !== undefined) {
              for (const prop in item.category) {
                for (const val in this.filters) {
                  if (item.category[prop].slug.includes(this.filters[val])) {
                    return item
                  }
                }
              }
            }
          }
        })
      }
      // Filter on start by most recent
      if (relCards.length > 4) {
        var count = 0
        relCards = relCards.filter((item) => {
          count++
          if (count < 4) {
            return item
          }
        })
      }
      return relCards
    }
  },
  watch: {
    '$route.query': {
      immediate: true,
      handler () {
        // this.post[0].page_title = n.myProp
      }
    }
  },
  components: {
  },
  methods: {
    async getSinglePost () {
      var pathArray = window.location.href.split('/resource/')
      var idArray = pathArray[1].split('?')
      var id = idArray[0]

      try {
        const url = 'https://awrproject.com.au/wp-json/resources/resource?postid=' + id
        const response = await axios.get(url)
        const postData = response.data
        // this.$router.push('/resource/' + objResources[0].ID)
        this.post = postData
        this.loaded = true
      } catch (err) {
        if (err.response) {
          // client received an error response (5xx, 4xx)
          console.log('Server Error:', err)
        } else if (err.request) {
          // client never received a response, or request never left
          console.log('Network Error:', err)
        } else {
          console.log('Client Error:', err)
        }
      }
    },
    getPostFromThisCollection () {
      var collection = this.collection
      var pathArray = window.location.href.split('/resource/')
      // console.log('pathArray', pathArray)
      var indArray = pathArray[1].split('/')
      var theId = parseInt(indArray[0])
      // theId = theId * 1
      this.id = theId
      if (collection.length > 0) {
        for (const prop in collection) {
          if (collection[prop].ID.toString().includes(theId)) {
            this.post[0] = collection[prop]
          }
        }
        this.loaded = true
      } else {
        this.getSinglePost()
      }
    },
    setThisTag (slug, e) {
      const newSetTag = this.tagFilters
      newSetTag.splice(0)
      newSetTag.push(slug)
      // console.log('newSetTag', newSetTag)
      return newSetTag
    },
    gotoLink: function (id, togglecards) {
      this.getURLVariables()
      var link = '/resource/' + id + '?s=' + this.searchTerm + '&f=' + this.filters + '&t=' + this.toggleCards
      // refresh data on page
      this.getPostFromThisCollection(this.collection)
      return link
    },
    methodThatForcesUpdate () {
      this.$forceUpdate() // Notice we have to use a $ here
    },
    getURLVariables () {
      this.searchTerm = this.$route.query.s
      this.toggleCards = this.$route.query.t
      var f = this.$route.query.f
      var filtersArray = f.split(',')
      // console.log(filtersArray)
      if (filtersArray.length > 0) {
        this.filters = filtersArray
      }
    },
    getRoute () {
      var q = this.$route.fullPath
      // console.log('q', q)
      var query = ''
      if (q.indexOf('?')) {
        var queryArray = q.split('?')
        query = queryArray[1]
      }
      return query
    },
    getTheId () {
      var pathArray = window.location.href.split('/resource/')
      var indArray = pathArray[1].split('/')
      var theId = parseInt(indArray[0])
      return theId
    },
    fileSize: function (bytes) {
      var sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB']
      if (bytes === 0) return '0 Byte'
      var i = parseInt(Math.floor(Math.log(bytes) / Math.log(1024)))
      return Math.round(bytes / Math.pow(1024, i), 2) + ' ' + sizes[i]
    },
    zeroFunction (object) {
      if (object === 'object') {
        object = 'x'
      }
    }
  },
  mounted () {
  },
  created () {
    // set the post
    this.getURLVariables()
    if (this.collection.length > 0) {
      this.getPostFromThisCollection()
    } else {
      this.post = this.tmpPost
      this.getSinglePost()
    }
  }
}
</script>
