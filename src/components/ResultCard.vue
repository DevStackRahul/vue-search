<template>
  <div class="card" :class="'card-' + (index + 1)">
    <div class="card-content">
      <span v-if="item.post_tag[0].slug === 'simulations' || item.post_tag[0].slug === 'training'" class="icon-lock"><i class="icofont-ui-lock"></i></span>
      <div class="thumb" v-bind:class="item.post_tag[0].slug">
        <img v-if="item.thumbnail" class="img-thumb"
          :src="item.thumbnail.url"
          :alt="item.post_title"
        />
        <img v-else-if="item.featured_image" class="img-featured"
          :src="item.featured_image"
          :alt="item.post_title"
        />
        <!-- <div v-if="item.file" class="resource-thumb">
          <i class="icofont-file-pdf" />
          <img src="https://awrproject.com.au/wp-content/uploads/logo-1.png" alt="AWR">
        </div> -->
      </div>
      <p class="card-type">{{item.post_type}}</p>
      <h3>
      <router-link :to="gotoLink(item.ID, searchValue, tagFilters, toggleCards)" >
        {{ item.post_title }}
      </router-link>
      </h3>
      <p class="desc">
        <!-- <span v-for="x in tagFilters" :key="x">
          {{x}}
        </span> -->
      </p>
      <p class="tags" v-if="item.category || item.post_tags" >
        <a v-for="c in item.category" :key="c.slug">
          <span @click="setTag(c.slug, $event)" >#{{c.name.toLowerCase()}}</span>
        </a>
        <a v-for="t in item.post_tag" :key="t.slug">
          <span @click="setTag(t.slug, $event)" >#{{t.name.toLowerCase()}}</span>
        </a>
      </p>
    </div>
  </div>
</template>
<script>
export default {
  props: ['collection', 'item', 'index', 'tagFilters', 'setTag', 'route', 'searchValue', 'toggleCards', 'saveFilteredCards'],
  data () {
    return {
    }
  },
  methods: {
    methodThatForcesUpdate () {
      this.$forceUpdate() // Notice we have to use a $ here
    },
    gotoLink: function (id, searchterm, filters, togglecards) {
      var link = '/resource/' + id + '?s=' + searchterm + '&f=' + filters + '&t=' + togglecards
      return link
    },
    get150Thumbnail: function (filename) {
      var fileDotSplit = filename.split('.')
      var fileSlashSplit = fileDotSplit[fileDotSplit.length - 2].split('/')
      // console.log(fileSlashSplit)
      var thumbFileName = fileSlashSplit[fileSlashSplit.length - 1] + '-150x150' + '.' + fileDotSplit[fileDotSplit.length - 1]
      return thumbFileName
    },
    fileSize: function (bytes) {
      var sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB']
      if (bytes === 0) return '0 Byte'
      var i = parseInt(Math.floor(Math.log(bytes) / Math.log(1024)))
      return Math.round(bytes / Math.pow(1024, i), 2) + ' ' + sizes[i]
    }
  }
}
</script>
