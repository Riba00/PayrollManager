<template>
  <form
    @submit.prevent="onChangePassword"
    class="ui form error change-password"
  >
    <div class="three fields">
      <div class="field">
        <input
          type="password"
          placeholder="Current Password"
          v-model="formData.password"
          :class="{ error: formError.password }"
        />
      </div>

      <div class="field">
        <input
          type="password"
          placeholder="New Password"
          v-model="formData.newPassword"
          :class="{ error: formError.newPassword }"
        />
      </div>

      <div class="field">
        <input
          type="password"
          placeholder="Confirm New Password"
          v-model="formData.newPasswordConfirmation"
          :class="{ error: formError.newPasswordConfirmation }"
        />
      </div>
    </div>
    <div class="ui error message" v-if="messageError">
      <p class="header">{{ messageError }}</p>
    </div>
    <button
      class="ui button primary"
      type="submit"
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
import { reauthenticate } from "../../utils/firebaseFunctions";
import Swal from "sweetalert2";

export default {
  name: "ChangePassword",
  setup() {
    let formData = {};
    let formError = ref({});
    let messageError = ref("");
    let isLoading = ref(false);

    const schemaForm = Yup.object().shape({
      password: Yup.string().required(true),
      newPassword: Yup.string().required(true),
      newPasswordConfirmation: Yup.string()
        .required(true)
        .oneOf([Yup.ref("newPassword")], true),
    });

    const onChangePassword = async () => {
      formError.value = {};
      isLoading.value = true;
      messageError.value = "";
      try {
        await schemaForm.validate(formData, { abortEarly: false });
        try {
          const { password, newPassword } = formData;
          await reauthenticate(password);
          await auth.currentUser.updatePassword(newPassword);

          Swal.fire({
            title: "Password changed",
            text: "Please login again",
            icon: "success",
          });

          auth.signOut();
        } catch (error) {
          console.log(error);

          let firebaseMessage = JSON.parse(error.message).error.message;

          switch (firebaseMessage) {
            case "INVALID_LOGIN_CREDENTIALS":
              messageError.value = "Incorrect Current Password";
              break;
            default:
              messageError.value = firebaseMessage;
              break;
          }
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
      onChangePassword,
      isLoading,
      messageError,
    };
  },
};
</script>

<style lang="scss" scoped>
.ui.form.change-password {
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
