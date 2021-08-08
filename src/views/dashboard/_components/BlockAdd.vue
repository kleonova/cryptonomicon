<template>
  <section>
    <div class="flex">
      <div class="max-w-xs">
        <label for="ticker" class="block text-sm font-medium text-gray-700">Тикер</label>

        <div class="mt-1 relative rounded-md shadow-md">
          <input
            v-model="ticker"
            @keydown.enter="addTicker"
            type="text"
            name="ticker"
            id="ticker"
            class="
              block
              w-full
              p-2
              pr-10
              border-gray-300
              text-gray-900
              focus:outline-none focus:ring-gray-500 focus:border-gray-500
              sm:text-sm
              rounded-md
            "
            autocomplete="off"
            placeholder="Например DOGE"
          />
        </div>

        <!-- autocomplete -->
        <div
          v-if="ticker && filteredCoinList.length"
          class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
        >
          <span
            v-for="(coin, coinInd) in filteredCoinList"
            :key="coinInd"
            class="
              inline-flex
              items-center
              px-2
              m-1
              rounded-md
              text-xs
              font-medium
              bg-gray-300
              text-gray-800
              cursor-pointer
            "
            @click="selectAutoTicker(coin)"
          >
            {{ coin.Symbol }}
          </span>
        </div>

        <!-- clue -->
        <div v-if="hasDuplicate" class="text-sm text-red-600">Такой тикер уже добавлен</div>
      </div>
    </div>
    <button-add :disabled="!ticker || hasDuplicate" @click="addTicker" class="my-4" />
    <hr class="w-full border-t border-gray-600 my-4" />
  </section>
</template>

<script>
  import ButtonAdd from '@/components/buttons/ButtonAdd'

  export default {
    name: 'BlockAdd',
    components: {
      ButtonAdd,
    },
    props: {
      tickers: {
        type: Array,
        required: false,
        default: () => [],
      },
    },
    emits: {
      add: (value) => {
        return typeof value === 'string' && value.length > 0
      },
    },
    data() {
      return {
        ticker: '',
        coinList: [],
      }
    },
    computed: {
      filteredCoinList() {
        const matchedCoinList = []
        const re = new RegExp(this.ticker.toLowerCase(), 'i')

        this.coinList.forEach((coin) => {
          const found = coin.Symbol.toLowerCase().match(re)
          if (found) {
            matchedCoinList.push({
              ...coin,
              found,
            })
          }
        })

        matchedCoinList.sort((a, b) => {
          if (a.found.index === b.found.index) {
            return a.Symbol > b.Symbol ? 1 : -1
          } else {
            return a.found.index > b.found.index ? 1 : -1
          }
        })

        return matchedCoinList.slice(0, 4)
      },
      hasDuplicate() {
        return !!this.tickers.find(({ name }) => name.toLowerCase() === this.ticker.toLowerCase())
      },
    },
    watch: {},
    created() {
      this.getCoinList()
    },
    methods: {
      async getCoinList() {
        // Получить все доступные валюты
        // todo fix получения монеток
        const fetchResponse = await fetch(
          `https://min-api.cryptocompare.com/data/all/coinlist?summary=true&api_key=860436f656fb823e6b7e1d8a47e92f633c2b1c53b97538cd723426e4db835500`
        )

        const data = await fetchResponse.json()
        if (data && data?.Data) {
          this.coinList = Object.values(data.Data)
        } else {
          // todo исправить вывод
          console.error('Ошибка при получении данных')
        }
      },
      selectAutoTicker(autoTicker) {
        this.ticker = autoTicker.Symbol

        if (!this.hasDuplicate) {
          this.addTicker()
        }
      },
      addTicker() {
        if (!this.ticker) return

        this.$emit('add', this.ticker)

        this.ticker = ''
      },
    },
  }
</script>
