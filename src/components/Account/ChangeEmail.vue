<template>
  <form class="ui form error change-email" @submit.prevent="onChangeEmail">
    <div class="two fields">
      <div class="field">
        <input
          type="text"
          placeholder="New Email"
          v-model="formData.email"
          :class="{ error: formError.email }"
        />
      </div>
      <div class="field">
        <input
          type="password"
          placeholder="Password"
          v-model="formData.password"
          :class="{ error: formError.password }"
        />
      </div>
    </div>
    <div class="ui error message" v-if="messageError">
      <p class="header">{{ messageError }}</p>
    </div>
    <button
      type="submit"
      class="ui button primary"
      :class="{ loading: isLoading }"
    >
      Save
    </button>
  </form>
</template>

<script>
import { ref } from "vue";
import * as Yup from "yup";
import { reauthenticate } from "../../utils/firebaseFunctions";
import { auth } from "../../utils/firebase";
import Swal from "sweetalert2";

export default {
  title: "ChangeEmail",
  setup() {
    let formData = {};
    let formError = ref({});
    let messageError = ref("");
    let isLoading = ref(false);

    const schemaForm = Yup.object().shape({
      email: Yup.string().email(true).required(true),
      password: Yup.string().required(true),
    });

    const onChangeEmail = async () => {
      formError.value = {};
      isLoading.value = true;
      messageError.value = "";

      try {
        await schemaForm.validate(formData, { abortEarly: false });
        try {
          const { email, password } = formData;
          await reauthenticate(password);
          await auth.currentUser.verifyBeforeUpdateEmail(email);
          Swal.fire({
            title: "Email not changed yet...",
            text: "Please check your email to confirm the change",
            icon: "success",
          });
          auth.signOut();
        } catch (error) {
          console.log(error);
          messageError.value = error.message;
        }
      } catch (error) {
        error.inner.forEach((err) => {
          formError.value[err.path] = err.message;
        });
      }
      isLoading.value = false;
    };

    return {
      formData,
      formError,
      messageError,
      onChangeEmail,
      isLoading,
    };
  },
};
</script>

<style lang="scss" scoped>
.ui.form.change-email {
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
