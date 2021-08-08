<template>
  <div class="modal" @click="clickOutside">
    <div class="modal__content" @click.stop>
      <div class="modal__header">
        <h3 class="text-xl font-semibold">{{ title }}</h3>
        <button @click="close">x</button>
      </div>

      <div class="modal__body">
        <slot />
      </div>

      <div class="modal__footer">
        <slot name="actions" :close="close" :confirm="confirm">
          <button class="text-white bg-gray-600 rounded-full py-2 px-4" @click="confirm">Ок</button>
          <button class="text-gray-500 py-2 px-4 border rounded-full ml-2" @click="close">
            Отмена
          </button>
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'ModalWindow',
    props: {
      targetId: {
        type: String,
        required: false,
        default: '',
      },
      title: {
        type: String,
        required: false,
        default: '',
      },
    },
    emits: {
      close: null,
      confirm: null,
    },
    data() {
      return {}
    },
    mounted() {
      document.addEventListener('keydown', this.handleKeydown)
    },
    beforeUnmount() {
      document.removeEventListener('keydown', this.handleKeydown)
    },
    methods: {
      confirm() {
        this.$emit('confirm')
        this.close()
      },
      close() {
        this.$emit('close')
      },
      clickOutside() {
        this.close()
      },
      handleKeydown(e) {
        if (e.key === 'Escape') {
          this.close()
        }
      },
    },
  }
</script>

<style lang="scss">
  .modal {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0;
    padding: 0 0 150px 0;
    background: rgba(0, 0, 0, 0.5); /* цвет фона */
    z-index: 100;

    &__content {
      background-color: white;
      padding: 30px;
      border-radius: 15px;
      min-width: 320px;
      max-width: 90vw;
    }

    &__header {
      display: flex;
      align-items: baseline;
      justify-content: space-between;
      margin-bottom: 20px;
    }

    &__footer {
      display: flex;
      align-items: baseline;
      justify-content: flex-end;
      margin-top: 20px;
    }
  }
</style>
