<template>
  <div class="auth">
    <Login v-if="showLogin" :changeForm="changeForm" />
    <Register v-if="!showLogin" :changeForm="changeForm" />
  </div>
</template>

<script>
import { ref } from "vue";
import Login from "../components/Auth/Login";
import Register from "../components/Auth/Register";

export default {
  name: "Auth",
  components: {
    Login,
    Register,
  },
  setup() {
    const showLogin = ref(true);

    const changeForm = () => {
      showLogin.value = !showLogin.value;
    };

    return {
      showLogin,
      changeForm,
    };
  },
};
</script>

<style lang="scss" scoped>
.auth {
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgb(4, 78, 157);
  background: linear-gradient(
    0deg,
    rgba(4, 78, 157, 1) 0%,
    rgba(0, 174, 255, 1) 100%
  );
}
</style>
