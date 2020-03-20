<template>
  <div class="client-root">
    <div class="column is-four-fifths">
      <Action
        :outcome="diceOutcome"
        :players="players"
        :currentPlayer="currentPlayer"
      />
      <Dice
        :outcome="diceOutcome"
        :rolling="diceRolling"
        :currentPlayer="currentPlayer"
        @roll="roll" />
      <Ready
        :currentPlayer="currentPlayer"
        :outcome="diceOutcome"
        @click="nextPlayer"
      />
    </div>
    <div class="column">
      <PlayerList
        :players="players"
        :currentPlayer="currentPlayer"
      />
    </div>
  </div>
</template>
<style>
.client-root {
  display: flex;
  width: 100%;
  margin-top: 0;
  height: 100%;
}
</style>
<script>
import Peer from 'peerjs';
import Action from '@/components/Action.vue';
import Dice from '@/components/Dice.vue';
import Ready from '@/components/Ready.vue';
import PlayerList from '@/components/PlayerList.vue';

export default {
  data() {
    return {
      diceRolling: false,
      diceOutcome: [],
      currentPlayer: null,
      serverID: '',
      serverConnection: null,
      peer: {},
      players: [],
    };
  },
  methods: {
    nextPlayer() {
      this.serverConnection.send({ type: 'NEXT_PLAYER' });
    },
    roll() {
      this.serverConnection.send({ type: 'DICE_ROLL' });
    },
    onPlayerJoin(data) {
      if (!this.players.includes(data.name)) {
        this.players = [...this.players, data.name];
      }
    },
    onPlayerLeft(data) {
      this.players = this.players.filter((p) => p !== data.name);
    },
    onDiceRollStart(data) {
      this.diceRolling = true;
      this.currentPlayer = data.name;
    },
    onDiceRollEnd(data) {
      this.diceRolling = false;
      this.diceOutcome = data.outcome;
      this.currentPlayer = data.name;
    },
    onDataSync(data) {
      this.diceRolling = data.diceRolling;
      this.diceOutcome = data.diceOutcome;
      this.currentPlayer = data.currentPlayer;
      this.players = data.players;
    },
    onNextPlayer(data) {
      this.diceOutcome = [];
      this.diceState = null;
      this.currentPlayer = data.name;
    },
    onMessageReceived(data) {
      if (data.type === 'PLAYER_JOINED') this.onPlayerJoin(data);
      if (data.type === 'PLAYER_LEFT') this.onPlayerLeft(data);
      if (data.type === 'DICE_ROLL_START') this.onDiceRollStart(data);
      if (data.type === 'DICE_ROLL_END') this.onDiceRollEnd(data);
      if (data.type === 'NEXT_PLAYER') this.onNextPlayer(data);
      if (data.type === 'DATA_SYNC') this.onDataSync(data);
    },
    sendMessage(data) {
      this.serverConnection.send(data);
    },
  },
  created() {
    this.serverID = this.$route.query.serverID;
    this.peer = new Peer(null, { debug: 3 });
    this.peer.on('open', () => {
      this.serverConnection = this.peer.connect(this.serverID, { reliable: true });
      this.serverConnection.on('open', () => {
        this.serverConnection.send({ name: window.localStorage.getItem('name'), type: 'PLAYER_JOIN' });
        this.serverConnection.on('data', this.onMessageReceived);
      });
    });
    this.peer.on('error', console.error);
  },
  components: {
    Action,
    Dice,
    PlayerList,
    Ready,
  },
};
</script>
