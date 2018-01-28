<template>
  <div>
    <button @click="subscribe">I want to know the winner!</button>
    <h2>Participantes</h2>
    <ul v-if="members.length > 0">
      <li v-for="member in members">
        <img :src="member.photo.thumb_link" :alt="`Pic of ${member.name}`">
        <h3>{{ member.name }}</h3>
      </li>
    </ul>
  </div>
</template>

<script>
import runtime from 'serviceworker-webpack-plugin/lib/runtime';
import { firebaseApp } from '../firebase';

export default {
  name: 'members',
  computed: {
    members() {
      return this.$store.state.members;
    },
  },
  created() {
    this.registerServiceWorker();
  },
  mounted() {
    this.$store.dispatch('LOAD_MEMBERS_LIST');
    firebaseApp.messaging().onTokenRefresh(this.handleTokenRefresh);
  },
  methods: {
    registerServiceWorker() {
      if ('serviceWorker' in navigator) {
        // eslint-disable-next-line
        const registration = runtime.register();

        navigator.serviceWorker.ready
          .then((serviceWorkerRegistration) => {
            firebaseApp.messaging().useServiceWorker(serviceWorkerRegistration);
          });
      }
    },
    subscribe() {
      console.log('subscribe');
      firebaseApp.messaging()
        .requestPermission()
        .then(() => { this.handleTokenRefresh(); });
    },
    handleTokenRefresh() {
      return firebaseApp.messaging()
        .getToken()
        .then((token) => {
          console.log(token);
          firebaseApp.database().ref('/tokens').push({
            token,
          });
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
  display: table;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

li {
  display: block;
  margin: 10px 0;
  width: 50%;
  float: left;
  border: 1px solid #ccc;
  padding: 10px;
  min-height: 102px;
}

li img {
  height: 100%;
  width: auto;
  float: left;
  margin: 0 10px 0 0;
}

a {
  color: #35495E;
}
</style>
