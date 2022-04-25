<template>
  <div class="results">
    <h2>Latest resources</h2>
    <div class="cards" style="min-width: 500px;">

    <div v-if="!loaded" class="loading">
    Loading resources...
      <div class="lds-ring"><div></div><div></div><div></div><div></div></div>
    </div>

      <ResultCard
        v-for="(item, index) in collection"
        class="card-content"
        :key="item.ID"
        :index="index"
        :item="item"
        :collection="collection"
      />
    </div>
  </div>
</template>

<script>
import ResultCard from '@/components/ResultCard.vue'

export default {
  props: ['collection', 'loaded'],
  data () {
    return {
    }
  },
  components: {
    ResultCard
  }
}
</script>
