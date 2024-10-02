<template>
  <div class="register">
    <h1>Create Account</h1>

    <form class="ui form" @submit.prevent="onRegister">
      <div class="field">
        <input
          type="email"
          placeholder="Email"
          v-model="formData.email"
          :class="{ error: formError.email }"
        />
        <p v-if="formError.email">{{ formError.email }}</p>
      </div>
      <div class="field">
        <input
          type="password"
          placeholder="Password"
          v-model="formData.password"
          :class="{ error: formError.password }"
        />
        <p v-if="formError.password">{{ formError.password }}</p>
      </div>
      <div class="field">
        <input
          type="password"
          placeholder="Confirm Password"
          v-model="formData.passwordConfirmation"
          :class="{ error: formError.passwordConfirmation }"
        />
        <p v-if="formError.passwordConfirmation">{{ formError.passwordConfirmation }}</p>
      </div>

      <button type="submit" class="ui button positive fluid" :class="{ 'loading': isLoading }">Register</button>
    </form>

    <p @click="changeForm">Login</p>
  </div>
</template>

<script>
import { ref } from "vue";
import { auth } from '../../utils/firebase';
import * as Yup from "yup";

export default {
  name: "Register",
  props: {
    changeForm: Function,
  },
  setup() {
    let formData = ref({
      email: "",
      password: "",
      passwordConfirmation: "",
    });

    let formError = ref({});
    let isLoading = ref(false);

    // Esquema de validación usando Yup
    const schemaForm = Yup.object().shape({
      email: Yup.string().email("Email is not valid").required("Email is required"),
      password: Yup.string().required("Password is required"),
      passwordConfirmation: Yup.string()
        .required("Password confirmation is required")
        .oneOf([Yup.ref("password")], "Passwords must match"),
    });

    // Función de registro
    const onRegister = async () => {
      isLoading.value = true;
      formError.value = {}; // Limpiar errores previos

      try {
        // Validar los datos del formulario
        await schemaForm.validate(formData.value, { abortEarly: false });

        // Intentar crear un usuario con Firebase Auth
        const { email, password } = formData.value;
        await auth.createUserWithEmailAndPassword(email, password);

      } catch (validationError) {
        if (validationError.inner) {
          validationError.inner.forEach((error) => {
            formError.value[error.path] = error.message;
          });
        } else {
          console.error("Firebase Auth Error:", validationError);
          formError.value.general = "An error occurred while creating the account.";
        }
      }
      isLoading.value = false;
    };

    return {
      formData,
      formError,
      onRegister,
      isLoading
    };
  },
};
</script>

<style lang="scss" scoped>
.register {
  background-color: #fff;
  padding: 30px;
  box-shadow: 0px 0px 38px -5px rgba(0, 0, 0, 0.45);
  width: 400px;
  border-radius: 10px;

  h1 {
    text-align: center;
    margin-bottom: 30px;
  }

  form {
    input.error {
      border-color: #faa;
      background-color: #ffeded;
    }

    p {
      color: red;
      font-size: 0.875rem;
      margin: 5px 0 0 0;
    }
  }

  p {
    text-align: center;
    margin-top: 30px;

    &:hover {
      cursor: pointer;
      opacity: 0.6;
    }
  }
}
</style>
