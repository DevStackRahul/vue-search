<template>
    <div class="container">
      <div class="row">
        <div class="resource-hub">
          <h1 class="page-title">Resources</h1>
          <ul v-for="item in result" class="list-group" :key="item.model.id">
          <!-- list items from the result array -->
              <li class="list-group-item">Brand: {{item.brand}}</li>
              <li class="list-group-item">Id: {{item.model.id}}</li>
              <li class="list-group-item">Model: {{item.model.name}}</li>
          </ul>
          <div class="search-criteria" :class="{'filters-show': filtersShow }">
            <fieldset id="fieldset-search">
              <input type="text" id="search-input" v-on:keyup.enter="onEnter" v-model="searchValue" placeholder="Search resources..." />
              <i v-if="searchValue" class="icofont-close-line close" v-on:click="clearSearch()" />
              <span>
                <i class="icofont-search-1"></i>
              </span>
            </fieldset>
            <div class="tag-wrapper">
              <h3 class="filters" v-on:click="toggleFilters()">Filters
                <i class="icofont-close-line close" />
              </h3>
              <div class="filterset filterset-1 col-lg-2">
                <fieldset :class="{'fieldset-toggle-on': toggleFieldsetFor }">
                  <legend v-on:click="toggleFielset('for')">For <span v-if="this.selectedCheckboxesFor > 0"><b>{{this.selectedCheckboxesFor}}</b></span></legend>
                  <span class="check-wrapper" v-for="cat in this.categoriesFor" :key="cat.term_id">
                    <input
                      :id="'chck-cat-' + cat.term_id"
                      type="checkbox"
                      name="tags[]"
                      class="vis-hidden new-post-tags"
                      :value="cat.slug"
                      v-model="tagFilters"
                    />
                    <label :for="'chck-cat-' + cat.term_id">{{cat.name}}</label>
                  </span>
                </fieldset>
              </div>
              <div class="filterset filterset-2 col-lg-6">
                <fieldset :class="{'fieldset-toggle-on': toggleFieldsetTopic }">
                  <legend v-on:click="toggleFielset('topic')">Topic <span v-if="this.selectedCheckboxesTopic > 0"><b>{{this.selectedCheckboxesTopic}}</b></span></legend>
                  <span class="check-wrapper" v-for="cat in this.categoriesTopic" :key="cat.term_id">
                    <input
                      :id="'chck-cat-' + cat.term_id"
                      type="checkbox"
                      name="tags[]"
                      class="vis-hidden new-post-tags"
                      :value="cat.slug"
                      v-model="tagFilters"
                    />
                    <label :for="'chck-cat-' + cat.term_id">{{cat.name}}</label>
                  </span>
                </fieldset>
              </div>
              <div class="filterset filterset-3 col-lg-4">
                <fieldset :class="{'fieldset-toggle-on': toggleFieldsetType }">
                  <legend v-on:click="toggleFielset('type')">Type <span v-if="this.selectedCheckboxesType > 0"><b>{{this.selectedCheckboxesType}}</b></span></legend>
                  <span class="check-wrapper" v-for="tag in this.tags" :key="tag.term_id">
                    <input
                      :id="'chck-tags-' + tag.term_id"
                      type="checkbox"
                      name="tags[]"
                      class="vis-hidden new-post-tags"
                      :value="tag.slug"
                      v-model="tagFilters"
                    />
                    <label :for="'chck-tags-' + tag.term_id">{{tag.name}}</label>
                  </span>
                </fieldset>
              </div>
            </div>
            <p class="clear-tags"><span class="clear" @click="clearFilters()" v-if="tagFilters.length > 0">Clear filters</span></p>
          </div>
          <div class="results">
            <div class="toggle-view">
                <div class="toggle-option toggle-cards" v-on:click="toggleResultCards(true)" :class="{ 'toggle-cards-on': toggleCards }">
                    <span class="line line-1"></span>
                    <span class="line line-2"></span>
                    <span class="line line-3"></span>
                    <span class="line line-4"></span>
                </div>
                <div class="toggle-option toggle-list" v-on:click="toggleResultCards(false)" :class="{ 'toggle-list-on': !toggleCards }">
                    <span class="line line-1"></span>
                    <span class="line line-2"></span>
                    <span class="line line-3"></span>
                </div>
            </div>
            <h2 v-if="(searchValue === null || searchValue === '') && tagFilters.length == 0">Latest resources...</h2>
            <h2 v-else>Results <span v-if="searchValue !== ''">for </span><b>{{searchValue}}</b><span v-if="tagFilters.length > 0"> with <b>{{ tagFilters.length }}</b> filter<span v-if="tagFilters.length > 1 || tagFilters.length === 0">s</span></span>...</h2>
            <div class="cards" :class="{ 'list-view': !toggleCards }">
              <div v-if="!loaded" class="loading">
                Loading resources...
                <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
              </div>
              <p class="no-results" v-if="filteredCards.length < 1 && loaded">{{noResultsMessage}}</p>
              <ResultCard v-for="(item, index) in filteredCards"
                class="card-outer"
                :key="item.ID"
                :index="index"
                :item="item"
                :tagFilters="tagFilters"
                :collection="collection"
                :setTag="setTag"
                :searchValue="searchValue"
                :toggleCards="toggleCards"
                :saveFilteredCards="this.filteredCards"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
import ResultCard from '@/components/ResultCard.vue'
import Fuse from 'fuse.js'

export default {
  name: 'Home',
  props: ['collection', 'loaded', 'tags', 'categoriesFor', 'categoriesTopic', 'filters'],
  components: {
    ResultCard
  },
  data () {
    return {
      ascending: true,
      sortBy: 'alphabetically',
      searchValue: '',
      noResultsMessage: 'There are no resources available with the current filters applied.',
      maxCookingTime: null,
      tagFilters: [],
      setEmptyResultsMessage: false,
      toggleCards: true,
      filtersShow: false,
      toggleFieldsetFor: true,
      toggleFieldsetTopic: true,
      toggleFieldsetType: true,
      selectedCheckboxesFor: 0,
      selectedCheckboxesTopic: 0,
      selectedCheckboxesType: 0,
      saveFilteredCards: this.filteredCards
    }
  },
  watch: {
    tagFilters: {
      handler: function () {
        this.addSelectedFilters()
      }
    },
    collection: {
      handler: function () {
        localStorage.setItem('resourceCollection', JSON.stringify(this.collection))
      }
    }
  },
  methods: {
    results (results) {
      this.result = results
    },
    addSelectedFilters () {
      var selectedCheckboxesFor = 0
      var selectedCheckboxesTopic = 0
      var selectedCheckboxesType = 0

      if (this.tagFilters.length > 0) {
        for (var item in this.tagFilters) {
          var tagFiltersSlug = this.tagFilters[item]
          for (var arrayItem in this.categoriesFor) {
            if (this.categoriesFor[arrayItem].slug === tagFiltersSlug) {
              selectedCheckboxesFor++
            }
          }
          for (var arrayItem2 in this.categoriesTopic) {
            if (this.categoriesTopic[arrayItem2].slug === tagFiltersSlug) {
              selectedCheckboxesTopic++
            }
          }
          for (var arrayItem3 in this.tags) {
            if (this.tags[arrayItem3].slug === tagFiltersSlug) {
              selectedCheckboxesType++
            }
          }
        }
      }
      this.selectedCheckboxesFor = selectedCheckboxesFor
      this.selectedCheckboxesTopic = selectedCheckboxesTopic
      this.selectedCheckboxesType = selectedCheckboxesType
    },
    toggleFilters () {
      this.addSelectedFilters()
      if (this.filtersShow) {
        this.filtersShow = false
      } else {
        this.filtersShow = true
      }
    },
    onEnter: function () {
      if (this.filtersShow) {
        this.filtersShow = false
      }
    },
    toggleFielset (which) {
      if (which === 'for') {
        if (this.toggleFieldsetFor) {
          this.toggleFieldsetFor = false
        } else {
          this.toggleFieldsetFor = true
        }
      } else if (which === 'topic') {
        if (this.toggleFieldsetTopic) {
          this.toggleFieldsetTopic = false
        } else {
          this.toggleFieldsetTopic = true
        }
      } else if (which === 'type') {
        if (this.toggleFieldsetType) {
          this.toggleFieldsetType = false
        } else {
          this.toggleFieldsetType = true
        }
      }
    },
    toggleResultCards (option) {
      if (option) {
        this.toggleCards = true
      } else {
        this.toggleCards = false
      }
    },
    setTag (slug, e) {
      const newSetTag = this.tagFilters
      newSetTag.splice(0)
      newSetTag.push(slug)
      // console.log('newSetTag', newSetTag)
      return newSetTag
    },
    clearFilters () {
      const tagFilter = this.tagFilters
      tagFilter.splice(0)
      this.clearQueryStrings()
      this.clearSearch()
      this.selectedCheckboxesFor = 0
      this.selectedCheckboxesTopic = 0
      this.selectedCheckboxesType = 0
      return tagFilter
    },
    clearSearch () {
      this.searchValue = null
      return this.searchValue
    },
    getQueryStrings () {
      // console.log('getQueryStrings')
      const queryString = window.location.href
      var queryStringSplitonHash = queryString.split('/#/')
      var queryStringFull = queryStringSplitonHash[1]
      var getQuerySFinal = ''
      if (queryStringFull !== '') {
        var getQueryS = queryStringFull.split('=')
        if (getQueryS[1].indexOf('&')) {
          var getQueryS2 = getQueryS[1].split('&')
          getQuerySFinal = getQueryS2[0]
        }
      }
      // console.log('getQuerySFinal', getQuerySFinal)
      const urlParams = new URLSearchParams(queryString)
      // var keyword = urlParams.get('s')
      var keyword = ''
      if (urlParams.get('s') !== '' && urlParams.get('s') !== null) {
        keyword = urlParams.get('s')
      } else {
        keyword = getQuerySFinal
      }
      // console.log('keyword', keyword)
      var filters = urlParams.get('f')
      var toggleCards = urlParams.get('t')
      // console.log(keyword, filters, toggleCards)
      if (keyword === 'null' || keyword === 'undefined') {
        this.searchValue = ''
      } else {
        this.searchValue = keyword
      }
      if (filters === '') {
        this.tagFilters = []
      }
      if (toggleCards === 'false') {
        this.toggleCards = false
      }
      if (filters !== '' && filters !== null) {
        var filtersArray = filters.split(',')
        for (var i = 0; i < filtersArray.length; i++) {
          this.tagFilters.push(filtersArray[i])
        }
      }
    },
    clearQueryStrings () {
      const queryString = window.location.href
      const urlParams = new URLSearchParams(queryString)
      urlParams.delete('s')
      urlParams.delete('f')
      urlParams.delete('t')
    },
    /*
   * Minimum window subsequence
   * @params {String} query: a query to search for
   * @params {String} strToSearch: a string to search in
   */
    minWinSeq (query, strToSearch) {
      let minSequence = null

      if (!query || !strToSearch) {
        return { minSequence: minSequence, positions: [] }
      }
      if (query.length > strToSearch.length) {
        return { minSequence: minSequence, positions: [] }
      }

      let end = 0
      let maxWinLength = strToSearch.length + 1
      let positions = []

      for (let i = 0, j = 0; i < strToSearch.length; i++) {
        if (strToSearch[i] === query[j]) {
          j += 1
          if (j >= query.length) {
            positions = []
            end = i + 1
            j -= 1
            while (j >= 0) {
              if (query[j] === strToSearch[i]) {
                positions.unshift(i)
                j -= 1
              }
              i -= 1
            }
            i += 1
            j += 1
            if (end - i < maxWinLength) {
              maxWinLength = end - i
              minSequence = strToSearch.substring(end, i)
            }
          }
        }
      }
      return { minSequence: minSequence, positions }
    },
    /*
    * Minimum window substring
    * @params {String} query: a query to search for
    * @params {String} searchString: a string to search in
    */
    minWinSub (query, searchString) {
      let minSequence = null
      let positions = []
      if (!query || !searchString) {
        return { minSequence, positions }
      }
      if (query.length > searchString.length) {
        return { minSequence, positions }
      }

      // build map containing letters and counts of those letters for our query
      const queryCharMap = new Map()
      query.forEach(letter => {
        const letterVal = queryCharMap.get(letter)
        if (letterVal) {
          queryCharMap.set(letter, letterVal + 1)
        } else {
          queryCharMap.set(letter, 1)
        }
      })

      const incrementMapKeyVal = (map, key, val) => {
        const currVal = map.get(key)
        if (currVal) {
          map.set(key, currVal + val)
        } else {
          map.set(key, 1)
        }
      }

      /*
        Loop over search string.  Move the right index until we satisfy.
        Compare against the current best.  Squeeze and compare until we
        no longer satsify and then start moving the right again
      */
      let left = 0
      let right = 0
      let frequenciesThatMatch = 0
      // const searchStringArr = searchString
      const windowCharMap = new Map()
      const totalFrequenciesToMatch = queryCharMap.size
      while (right < searchString.length) {
        const rightLetter = searchString[right]

        incrementMapKeyVal(windowCharMap, rightLetter, 1)

        if (queryCharMap.has(rightLetter)) {
          // do the frequencies of this letter in window and query match?
          if (
            queryCharMap.get(rightLetter) ===
            windowCharMap.get(rightLetter)
          ) {
            frequenciesThatMatch += 1
          }
        }

        while (
          frequenciesThatMatch === totalFrequenciesToMatch &&
          left <= right
        ) {
          minSequence = searchString // necessary to be here so we don't return the whole string if no match
          const leftLetter = searchString[left]
          const windowSize = right - left + 1

          if (windowSize < minSequence.length) {
            minSequence = searchString.substring(left, right + 1)
            positions = minSequence.reduce((acc, curr, idx) => {
              if (queryCharMap.has(curr)) {
                acc.push(left + idx)
              }
              return acc
            }, [])
          }

          // left will increase so decrement val for letter at that position in windowCharMap
          incrementMapKeyVal(windowCharMap, leftLetter, -1)

          // is left character a requirement
          if (queryCharMap.has(leftLetter)) {
            // does the letter frequency fail to meet the standard now>
            if (
              windowCharMap.get(leftLetter) <
              queryCharMap.get(leftLetter)
            ) {
              frequenciesThatMatch -= 1
            }
          }

          left += 1
        }
        right += 1
      }

      return { minSequence, positions }
    },
    /*
    * Sort items by rank, distance, or alphabetically
    */
    sortRankedItems (a, b) {
      if (a.rank === b.rank) {
        if (a.distance && a.distance !== b.distance) {
          return a.distance - b.distance
        }
        return a.text.localeCompare(b.text)
      }
      return b.rank - a.rank
    },
    /*
    * Generate an array filled from start..stop
    */
    range (start, stop) {
      return Array(Math.ceil((stop - start) / 1)).fill(start).map((x, y) => x + y * 1)
    }
  },
  computed: {
    filteredCards () {
      let tempCards = this.collection
      // Process search input
      if (this.searchValue !== '' && this.searchValue !== 'null' && this.searchValue) {
        const newresults = new Fuse(tempCards, { keys: ['post_title'] }).search(this.searchValue)
        tempCards = newresults.map((item) => {
          return item.item
        })
      }
      // Filter out tag
      if (this.tagFilters.length > 0 && this.tagFilters) {
        tempCards = tempCards.filter((item) => {
          if (item.post_tag !== undefined) {
            for (const prop in item.post_tag) {
              for (const val in this.tagFilters) {
                if (item.post_tag[prop].slug.includes(this.tagFilters[val])) {
                  return item
                }
              }
            }
          }
          if (item.category !== undefined) {
            for (const prop in item.category) {
              for (const val in this.tagFilters) {
                if (item.category[prop].slug.includes(this.tagFilters[val])) {
                  return item
                }
              }
            }
          }
        })
      }
      // Filter on start by most recent
      if ((this.searchValue === 'null' || this.searchValue === '') && this.tagFilters.length < 1) {
        var count = 0
        tempCards = tempCards.filter((item) => {
          count++
          if (count < 10) {
            return item
          }
        })
      }

      // Filter out by cooking time
      if (this.maxCookingTime) {
        tempCards = tempCards.filter((item) => {
          return (item.cookingTime <= this.maxCookingTime)
        })
      }
      // Sort by alphabetical order
      tempCards = tempCards.sort((a, b) => {
        if (this.sortBy === 'alphabetically') {
          const fa = a.post_title.toLowerCase()
          const fb = b.post_title.toLowerCase()

          if (fa < fb) {
            return -1
          }
          if (fa > fb) {
            return 1
          }
          return 0
          // Sort by cooking time
        } else if (this.sortBy === 'cookingTime') {
          return a.cookingTime - b.cookingTime
        }
      })
      // Show sorted array in descending or ascending order
      if (!this.ascending) {
        tempCards.reverse()
      }
      return tempCards
    }
  },
  mounted () {
    this.getQueryStrings()
  }
}
</script>
