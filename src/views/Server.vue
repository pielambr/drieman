<template>
  <div class="server-root">
    <div class="column is-four-fifths">
      <h1>Kopieer en deel code: <b>{{peer._id}}</b></h1>
      <Action
        :outcome="diceOutcome"
        :players="playerNames"
        :currentPlayer="currentPlayer"
      />
      <Dice
        :outcome="diceOutcome"
        :rolling="diceRolling"
        :currentPlayer="currentPlayer"
        @roll="onRoll" />
      <Ready
        :currentPlayer="currentPlayer"
        :outcome="diceOutcome"
        @click="onNextPlayer"
      />
    </div>
    <div class="column">
      <PlayerList
        :players="playerNames"
        :currentPlayer="currentPlayer"
      />
    </div>
  </div>
</template>
<style>
.server-root {
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
import PlayerList from '@/components/PlayerList.vue';
import Ready from '@/components/Ready.vue';

export default {
  data() {
    return {
      id: '',
      peer: {},
      diceRolling: false,
      diceOutcome: [],
      players: {},
      currentPlayer: null,
    };
  },
  computed: {
    playerNames() {
      return Object.keys(this.players);
    },
  },
  methods: {
    nextPlayer() {
      const idx = this.playerNames.indexOf(this.currentPlayer);
      if (idx === this.playerNames.length - 1) return this.playerNames[0];
      return this.playerNames[idx + 1];
    },
    onNextPlayer() {
      const nextPlayer = this.nextPlayer();
      this.currentPlayer = nextPlayer;
      this.diceOutcome = [];
      this.send({
        type: 'DATA_SYNC',
        diceOutcome: this.diceOutcome,
        diceRolling: this.diceRolling,
        players: this.playerNames,
        currentPlayer: this.currentPlayer,
      });
    },
    onPlayerJoin(connection, data) {
      if (!Object.keys(this.players).includes(data.name)) {
        this.$set(this.players, data.name, connection);
        this.send({ type: 'PLAYER_JOINED', name: data.name });
      }
    },
    onPlayerLeave(connection) {
      const name = this.playerNames.find((n) => {
        if (!this.players[n]) return false;
        return this.players[n].peer === connection.peer;
      });
      this.$delete(this.players, name);
      this.send({ type: 'PLAYER_LEFT', name });
    },
    onRoll() {
      this.diceRolling = true;
      this.diceOutcome = [];
      this.send({ type: 'DICE_ROLL_START', outcome: this.diceOutcome, name: this.currentPlayer });
      window.setTimeout(() => {
        this.diceOutcome = [Math.ceil(Math.random() * 6), Math.ceil(Math.random() * 6)];
        this.diceRolling = false;
        this.send({ type: 'DICE_ROLL_END', outcome: this.diceOutcome, name: this.currentPlayer });
      }, 2500);
    },
    send(data) {
      Object.values(this.players).forEach((connection) => {
        if (connection) connection.send(data);
      });
    },
  },
  created() {
    this.$set(this.players, window.localStorage.getItem('name'), null);
    this.currentPlayer = window.localStorage.getItem('name');
    this.peer = new Peer(null, { debug: 3 });
    this.peer.on('open', (id) => {
      this.id = id;
    });
    this.peer.on('connection', (conn) => {
      conn.on('data', (data) => {
        if (data.type === 'PLAYER_JOIN') this.onPlayerJoin(conn, data);
        if (data.type === 'DICE_ROLL') this.onRoll();
        if (data.type === 'NEXT_PLAYER') this.onNextPlayer();
        conn.send({
          type: 'DATA_SYNC',
          diceOutcome: this.diceOutcome,
          diceRolling: this.diceRolling,
          players: this.playerNames,
          currentPlayer: this.currentPlayer,
        });
      });
      conn.on('open', () => {
        conn.send('hello!');
      });
      conn.on('close', () => {
        this.onPlayerLeave(conn);
      });
    });
    this.peer.on('error', console.error); // eslint-disable-line
  },
  components: {
    Action,
    Dice,
    PlayerList,
    Ready,
  },
};
</script>
