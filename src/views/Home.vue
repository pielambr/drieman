<template>
  <div class="home-root">
    <div class="field">
      <div class="control">
        <input
          class="input"
          type="text"
          required
          placeholder="Nicknaam"
          @input="saveName"
        >
      </div>
    </div>
    <button class="button is-primary" @click="openServer" v-if="!!name">
      Maak een sessie
    </button>
    <button class="button is-primary is-light" @click="openClient" v-if="!!name">
      Join een sessie
    </button>
  </div>
</template>
<style scoped>
.home-root {
  margin-top: 100px;
}
.home-root > button + button {
  margin-left: 20px;
}
.home-root > .field {
  margin-left: auto;
  margin-right: auto;
  max-width: 300px;
}
</style>
<script>
export default {
  data() {
    return {
      name: '',
    };
  },
  methods: {
    checkName() {
      if (!window.localStorage.getItem('name')) {
        window.alert('Gelieve een nicknaam op te geven'); // eslint-disable-line
        return false;
      }
      return true;
    },
    openServer() {
      if (this.checkName()) this.$router.push('/server');
    },
    openClient() {
      if (this.checkName()) {
        const serverID = window.prompt('Wat is de server ID?'); // eslint-disable-line
        if (serverID) this.$router.push(`/client?serverID=${serverID}`);
      }
    },
    saveName({ target: { value } }) {
      this.name = value;
      window.localStorage.setItem('name', value);
    },
  },
};
</script>
