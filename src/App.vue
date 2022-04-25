<template>
  <router-view
    :collection="collection"
    :loaded="loaded"
    :tagFilters="tagFilters"
    :tags="tags" :categoriesFor="categoriesFor"
    :categoriesTopic="categoriesTopic"
    :saveFilteredCards="saveFilteredCards"
  />
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      collection: [],
      loaded: false,
      tags: [],
      categoriesFor: [],
      categoriesTopic: [],
      tagFilters: [],
      saveFilteredCards: []
    }
  },
  computed: {
  },
  methods: {
    async getPosts () {
      try {
        const url = 'https://awrproject.com.au/wp-json/resources/resources-route'
        const response = await axios.get(url)
        const objResources = response.data
        this.collection = objResources
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
      this.getTags()
      this.loaded = true
    },
    doesTagExist: function (item) {
      for (var i = 0; i < this.tags.length; i++) {
        if (this.tags[i].slug === item) {
          return true
        }
      }
      return false
    },
    doesCatExist: function (targetArr, item) {
      var arr = this.categoriesTopic
      if (targetArr === 'for') {
        arr = this.categoriesFor
      }
      for (var i = 0; i < arr.length; i++) {
        if (arr[i].slug === item) {
          return true
        }
      }
      return false
    },
    getTags () {
      let dataTags = this.collection
      dataTags = dataTags.filter((item) => {
        if (item.post_tag !== undefined) {
          for (const prop in item.post_tag) {
            if (this.doesTagExist(item.post_tag[prop].slug) === false) {
              this.tags.push(item.post_tag[prop])
            }
          }
        }
      })
      this.zeroFunction(dataTags)
      this.getCats()
    },
    zeroFunction (object) {
      if (object === 'object') {
        object = 'x'
      }
    },
    getCats () {
      let dataTags = this.collection
      dataTags = dataTags.filter((item) => {
        if (item.category !== undefined) {
          for (const prop in item.category) {
            if (item.category[prop].slug === 'employees' ||
                item.category[prop].slug === 'employers' ||
                item.category[prop].slug === 'individuals' ||
                item.category[prop].slug === 'leaders' ||
                item.category[prop].slug === 'organisations'
            ) {
              // check if exists and pop 'For' array
              if (this.doesCatExist('for', item.category[prop].slug) === false) {
                this.categoriesFor.push(item.category[prop])
              }
            } else {
              // check if exists and pop 'Topics' array
              if (this.doesCatExist('topic', item.category[prop].slug) === false) {
                this.categoriesTopic.push(item.category[prop])
              }
            }
          }
        }
      })
      this.zeroFunction(dataTags)
    }
  },
  created () {
    this.getPosts()
  }
}
</script>
