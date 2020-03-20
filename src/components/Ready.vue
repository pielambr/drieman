<template>
  <button
    class="button is-link ready"
    @click="$emit('click')"
    v-if="active"
  >
    Klaar met beurt
  </button>
</template>
<style>
.button.ready {
  margin-top: 40px;
}
</style>
<script>
export default {
  props: {
    currentPlayer: String,
    outcome: Array,
  },
  computed: {
    active() {
      return this.outcome
        && this.outcome.length === 2
        && window.localStorage.getItem('name') === this.currentPlayer;
    },
  },
};
</script>
