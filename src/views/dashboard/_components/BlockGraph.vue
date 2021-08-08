<template>
  <section class="relative">
    <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
      {{ selectedTicker.name }} - USD
    </h3>

    <div class="flex items-end border-gray-600 border-b border-l h-64" ref="graph">
      <div
        v-for="(bar, indexBar) in normalizedGraph"
        :key="indexBar"
        class="bg-purple-800 border"
        :style="`height: ${bar}%; width: ${widthColumn}px`"
      />
    </div>

    <button @click="closeGraph" type="button" class="absolute top-0 right-0">
      <signs-cross />
    </button>
  </section>

  <div class="mt-2">Не принимайте график на веру! Сбит масштаб.</div>
</template>

<script>
  import SignsCross from '@/components/signs/SignsCross'

  export default {
    name: 'BlockGraph',
    components: {
      SignsCross,
    },
    props: {
      selectedTicker: {
        type: Object,
        required: true,
        default: () => ({}),
      },
    },
    emits: {
      close: null,
    },
    data() {
      return {
        graph: [],
        maxGraphElements: 1,
        widthColumn: 30,
      }
    },
    computed: {
      normalizedGraph() {
        const maxValue = Math.max(...this.graph)
        const minValue = Math.min(...this.graph)

        if (maxValue === minValue) {
          return this.graph.map(() => 50)
        }

        return this.graph.map((value) => 5 + ((value - minValue) * 95) / (maxValue - minValue))
      },
    },
    watch: {
      selectedTicker() {
        this.graph = []
        this.$nextTick().then(this.calculateMaxGraphElements)
      },
      maxGraphElements() {
        this.cutGraph()
      },
    },
    mounted() {
      this.calculateMaxGraphElements()

      window.addEventListener('resize', this.calculateMaxGraphElements)
    },
    beforeUnmount() {
      window.removeEventListener('resize', this.calculateMaxGraphElements)
    },
    methods: {
      calculateMaxGraphElements() {
        const widthGraph = this.$refs.graph?.clientWidth
        this.maxGraphElements = widthGraph ? Math.floor(widthGraph / this.widthColumn) : 0
      },
      cutGraph() {
        if (this.graph.length > this.maxGraphElements) {
          this.graph = this.graph.slice(-1 * this.maxGraphElements)
        }
      },
      addColumn(price) {
        this.graph.push(price)
        this.cutGraph()
      },
      closeGraph() {
        this.$emit('close')
      },
    },
  }
</script>
