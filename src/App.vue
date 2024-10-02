<template>
  <template v-if="user">
    <router-view />
  </template>
  <div v-if="!user && user !== undefined">
    <Auth />
  </div>
  
</template>

<script>
import { onMounted, computed } from "vue";
import { useStore } from "vuex";
import { auth } from "./utils/firebase";
import Auth from './views/Auth'

export default {
  components: {
    Auth
  },
  setup() {
    const store = useStore();
    const user = computed(() => store.state.user);

    onMounted(() => {
      auth.onAuthStateChanged((user) => {
        store.commit("setUser", user);
      });
    });

    return {
      user,
    };
  },
};
</script>

<style></style>
