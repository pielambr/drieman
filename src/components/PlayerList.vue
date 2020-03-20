<template>
  <div class="playerlist-root">
    <div v-for="player in players" :key="player">
      <div :class="{bold: player === currentPlayer}">
        {{player}}
      </div>
    </div>
    <div v-if="!players.length">
      <em>Geen spelers</em>
    </div>
  </div>
</template>
<style>
.playerlist-root {
  height: 100%;
  border-left: 1px solid lightgrey;
}
.bold {
  font-weight: bold;
}
</style>
<script>
export default {
  props: {
    players: Array,
    currentPlayer: String,
  },
};
</script>
