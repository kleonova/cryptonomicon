<template>
  <div class="w-screen min-h-screen bg-gray-100">
    <div class="container mx-auto flex flex-col items-center p-4">
      <div class="container">
        <!-- field add -->
        <block-add :tickers="tickers" @add="addTicker" />

        <!-- filter -->
        <div class="my-4">
          <label for="filter" class="block text-sm font-medium text-gray-700">Фильтр</label>

          <div class="mt-1 relative rounded-md shadow-md">
            <input
              v-model="filter"
              id="filter"
              type="text"
              class="
                block
                w-full
                p-2
                border-gray-300
                text-gray-900
                focus:outline-none focus:ring-gray-500 focus:border-gray-500
                sm:text-sm
                rounded-md
              "
              autocomplete="off"
              placeholder=""
            />
          </div>

          <div class="my-4">
            <button
              class="
                inline-flex
                items-center
                py-2
                px-4
                border border-transparent
                shadow-sm
                text-sm
                leading-4
                font-medium
                rounded-full
                text-white
                bg-gray-600
                hover:bg-gray-700
                transition-colors
                duration-300
                focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500
                disabled:opacity-75
              "
              :disabled="page === 1"
              @click="setPage(page - 1)"
            >
              Назад
            </button>

            <span class="mx-2">{{ page }}</span>

            <button
              class="
                inline-flex
                items-center
                py-2
                px-4
                border border-transparent
                shadow-sm
                text-sm
                leading-4
                font-medium
                rounded-full
                text-white
                bg-gray-600
                hover:bg-gray-700
                transition-colors
                duration-300
                focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500
                disabled:opacity-75
              "
              :disabled="!hasNextPage"
              @click="setPage(page + 1)"
            >
              Вперед
            </button>
          </div>

          <hr class="w-full border-t border-gray-600 my-4" />
        </div>

        <!-- list tickers -->
        <template v-if="paginatedTickers.length">
          <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
            <div
              v-for="t in paginatedTickers"
              :key="t.name"
              @click="selectTicker(t)"
              class="bg-white overflow-hidden shadow rounded-lg cursor-pointer"
              :class="{ 'border-purple-800 border-solid border-2': selectedTicker === t }"
            >
              <div class="px-4 py-5 sm:p-6 text-center">
                <dt class="text-sm font-medium text-gray-500 truncate">{{ t.name }} - USD</dt>
                <dd class="mt-1 text-3xl font-semibold text-gray-900">
                  {{ formatPrice(t.price) }}
                </dd>
              </div>
              <div class="w-full border-t border-gray-200"></div>
              <button
                @click.stop="deleteTicker(t)"
                class="
                  flex
                  items-center
                  justify-center
                  font-medium
                  w-full
                  bg-gray-100
                  px-4
                  py-4
                  sm:px-6
                  text-md text-gray-500
                  hover:text-gray-600 hover:bg-gray-200
                  transition-all
                  focus:outline-none
                "
              >
                <signs-trash />
                Удалить
              </button>
            </div>
          </dl>
          <hr class="w-full border-t border-gray-600 my-4" />
        </template>

        <!-- graph -->
        <block-graph
          v-if="selectedTicker"
          ref="blockGraph"
          :selected-ticker="selectedTicker"
          @close="selectedTicker = null"
        />

        <button @click="isOpenModal = true">Открыть модальное</button>
      </div>
    </div>
  </div>

  <modal-window
    v-if="isOpenModal"
    title="Добавить тикер"
    @close="handlerCloseModal"
    @confirm="handlerConfirmModal"
  >
    Lorem ipsum dolor sit amet, consectetur adipisicing elit. Magnam, tempore!

    <template #actions="{ confirm }">
      <button class="text-white bg-green-600 rounded-full py-2 px-4" @click="confirm()">
        confirm
      </button>
    </template>
  </modal-window>
</template>

<script>
  import { subscribeToTicker, unsubscribeFromTicker } from './api'
  import SignsTrash from '@/components/signs/SignsTrash'
  import BlockAdd from '@/views/dashboard/_components/BlockAdd'
  import BlockGraph from '@/views/dashboard/_components/BlockGraph'
  import ModalWindow from '@/components/modal/ModalWindow'

  export default {
    name: 'App',
    components: {
      BlockAdd,
      BlockGraph,
      SignsTrash,
      ModalWindow,
    },
    data() {
      return {
        tickers: [],
        /* graph */
        selectedTicker: null,
        /* filter */
        page: 1,
        countOnPage: 6,
        filter: '',
        /* */
        isOpenModal: false,
      }
    },
    computed: {
      startIndex() {
        return (this.page - 1) * this.countOnPage
      },
      endIndex() {
        return this.page * this.countOnPage
      },
      filteredTickers() {
        return this.tickers.filter(({ name }) =>
          name.toLowerCase().includes(this.filter.toLowerCase())
        )
      },
      paginatedTickers() {
        return this.filteredTickers.slice(this.startIndex, this.endIndex)
      },
      hasNextPage() {
        return this.filteredTickers.length > this.endIndex
      },
      pageStateOptions() {
        return {
          filter: this.filter,
          page: this.page,
        }
      },
    },
    created() {
      this.getFilterState()
      this.getTickersFromLS()
    },
    watch: {
      tickers() {
        // сохранить в LS
        localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers))
      },
      pageStateOptions(value) {
        window.history.pushState(
          null,
          document.title,
          `${window.location.pathname}?filter=${value.filter}&page=${value.page}`
        )
      },
      paginatedTickers() {
        if (this.paginatedTickers.length === 0 && this.page > 1) {
          this.page -= 1
        }
      },
      filter() {
        this.setPage(1)
      },
    },
    methods: {
      /* common */
      formatPrice(price) {
        if (price === '-') {
          return price
        }

        const floatPrice = parseFloat(price)

        return floatPrice > 1 ? floatPrice.toFixed(2) : floatPrice.toPrecision(2)
      },
      /* list */
      getTickersFromLS() {
        const tickersData = localStorage.getItem('cryptonomicon-list')

        if (tickersData) {
          this.tickers = JSON.parse(tickersData)
          this.tickers.forEach((ticker) => {
            subscribeToTicker(ticker.name, (newPrice) => this.updateTicker(ticker.name, newPrice))
          })
        }
      },
      addTicker(ticker) {
        const currentTicker = {
          name: ticker,
          price: '-',
        }

        this.tickers = [...this.tickers, currentTicker]
        this.filter = ''
        subscribeToTicker(currentTicker.name, (newPrice) =>
          this.updateTicker(currentTicker.name, newPrice)
        )
      },
      updateTicker(tickerName, price) {
        this.tickers
          .filter((t) => t.name === tickerName)
          .forEach((t) => {
            if (t === this.selectedTicker) {
              // todo fix
              this.$refs.blockGraph.addColumn(price)
            }
            t.price = price
          })
      },
      deleteTicker(tickerToRemove) {
        this.tickers = this.tickers.filter((t) => t !== tickerToRemove)
        if (this.selectedTicker === tickerToRemove) {
          this.selectedTicker = null
        }
        unsubscribeFromTicker(tickerToRemove.name)
      },
      selectTicker(ticker) {
        this.selectedTicker = ticker
      },
      /* filter */
      getFilterState() {
        const windowData = Object.fromEntries(new URL(window.location).searchParams.entries())

        const VALID_KEYS = ['filter', 'page']

        VALID_KEYS.forEach((key) => {
          if (windowData[key]) {
            this[key] = windowData[key]
          }
        })
      },
      setPage(value) {
        this.page = value
      },
      /* modal */
      handlerCloseModal() {
        this.isOpenModal = false
      },
      handlerConfirmModal() {
        console.log('handlerConfirm')
      },
    },
  }
</script>
