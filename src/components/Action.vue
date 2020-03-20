<template>
  <div v-if="outcome && outcome.length === 2" class="title is-3">
    <div v-if="both(1)">
      Sali!
    </div>
    <div v-else-if="both(2)">
      2 uitdelen!
    </div>
    <div v-else-if="both(3)">
      Drieman!
    </div>
    <div v-else-if="both(4)">
      4 uitdelen!
    </div>
    <div v-else-if="both(5)">
      5 uitdelen!
    </div>
    <div v-else-if="both(6)">
      6 uitdelen!
    </div>
    <div v-else-if="sum(5)">
      Sociaal!
    </div>
    <div v-else-if="sum(6)">
      Dubbel volgende beurt!
    </div>
    <div v-else-if="sum(7)">
      {{previousPlayer()}} neemt 1 slok
    </div>
    <div v-else-if="sum(9)">
      {{nextPlayer()}} neemt 1 slok
    </div>
    <div v-else-if="sum(11)">
      Pint uit tegen volgende beurt
    </div>
  </div>
</template>
<script>
export default {
  props: {
    outcome: Array,
    players: Array,
    currentPlayer: String,
  },
  methods: {
    both(number) {
      return this.outcome[0] === number && this.outcome[1] === number;
    },
    sum(number) {
      return this.outcome[0] + this.outcome[1] === number;
    },
    previousPlayer() {
      const idx = this.players.indexOf(this.currentPlayer);
      if (idx === 0) return this.players[this.players.length - 1];
      return this.players[idx - 1];
    },
    nextPlayer() {
      const idx = this.players.indexOf(this.currentPlayer);
      if (idx === this.players.length - 1) return this.players[0];
      return this.players[idx + 1];
    },
  },
};
</script>
