<template>
  <div>
    <div class="dice-root container">
      <div class="dices columns" v-if="!rolling && outcome.length === 2">
        <div class="column card">
          <div class="title is-3">
            {{outcome[0]}}
          </div>
        </div>
        <div class="column card">
          <div class="title is-3">
            {{outcome[1]}}
          </div>
        </div>
      </div>
      <div class="dices-rolling" v-if="rolling">
        De teerlingen worden geworpen...
      </div>
      <div class="dices-unknown" v-if="!rolling && outcome.length !== 2">
        Wachten tot er gerold wordt...
      </div>
    </div>
    <button class="button is-primary" @click="onRoll" v-if="active">
      Rol
    </button>
  </div>
</template>
<style>
.dice-root {
  margin-top: 40px;
  margin-bottom: 40px;
}
</style>
<script>
export default {
  props: {
    currentPlayer: String,
    outcome: Array,
    rolling: Boolean,
  },
  computed: {
    active() {
      return window.localStorage.getItem('name') === this.currentPlayer
        && this.outcome.length !== 2
        && !this.rolling;
    },
  },
  methods: {
    onRoll() {
      this.$emit('roll');
    },
  },
};
</script>
