<template>
  <form class="ui form change-name" @submit.prevent="onChangeName">
    <input
      type="text"
      placeholder="Full Name"
      v-model="name"
      :class="{ error: formError }"
    />
    <button
      class="ui button primary"
      type="sumbit"
      :class="{ loading: isLoading }"
    >
      Save
    </button>
  </form>
</template>

<script>
import { ref } from "vue";
import * as Yup from "yup";
import { auth } from "../../utils/firebase";
import { useStore } from "vuex";
import Swal from "sweetalert2";

export default {
  name: "ChangeName",
  setup() {
    const store = useStore();

    let name = ref("");
    let formError = ref(false);
    let isLoading = ref(false);

    const shcemaForm = Yup.object().shape({
      name: Yup.string().min(4, true).required(true),
    });

    const onChangeName = async () => {
      formError.value = false;
      try {
        await shcemaForm.validate({ name: name.value }, { abortEarly: false });

        try {
          await auth.currentUser.updateProfile({
            displayName: name.value,
          });
          store.dispatch("reloadUser");

          Swal.fire({
            position: "center",
            icon: "success",
            title: "Name updated",
            showConfirmButton: false,
            timer: 1500,
          });
        } catch (error) {
          console.log(error);
        }
      } catch (error) {
        error.inner.forEach((error) => {
          formError.value = error.message;
        });
      }
    };

    return {
      name,
      onChangeName,
      formError,
      isLoading,
    };
  },
};
</script>

<style lang="scss" scoped>
.ui.form.change-name {
  text-align: center;

  input.error {
    border-color: #faa;
    background-color: #ffeded;
  }

  .ui.button {
    margin: 20px 0 0 0;
  }
}
</style>
