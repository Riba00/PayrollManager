<template>
  <div class="login">
    <h1>Log In</h1>

    <form class="ui form" @submit.prevent="onLogin">
      <div class="field">
        <input 
          type="email" 
          placeholder="Email" 
          v-model="formData.email" 
          :class="{ error: formError.email }"
        />
        <p v-if="formError.email" class="error-message">{{ formError.email }}</p>
      </div>
      <div class="field">
        <input
          type="password"
          placeholder="Password"
          v-model="formData.password"
          :class="{ error: formError.password }"
        />
        <p v-if="formError.password" class="error-message">{{ formError.password }}</p>
      </div>
      <p v-if="formError.general" class="error-message">{{ formError.general }}</p>

      <button 
        type="submit" 
        class="ui button positive fluid" 
        :class="{ 'loading': isLoading }" 
        :disabled="isLoading"
      >
        Log In
      </button>
    </form>

    <p @click="changeForm" class="form-switch">Create Account</p>
  </div>
</template>

<script>
import { ref } from "vue";
import * as Yup from "yup";
import { auth } from '../../utils/firebase';

export default {
  name: "Login",
  props: {
    changeForm: Function,
  },
  setup() {
    const formData = ref({
      email: "",
      password: "",
    });

    const formError = ref({});
    const isLoading = ref(false);

    const schemaForm = Yup.object().shape({
      email: Yup.string().email("Invalid email format").required("Email is required"),
      password: Yup.string().required("Password is required"),
    });

    const onLogin = async () => {
      isLoading.value = true;
      formError.value = {};

      try {
        // Validar los datos del formulario
        await schemaForm.validate(formData.value, { abortEarly: false });

        // Intentar iniciar sesión con Firebase Auth
        const { email, password } = formData.value;
        await auth.signInWithEmailAndPassword(email, password);
      } catch (err) {
        // Si es un error de validación de Yup
        if (err.inner) {
          err.inner.forEach((error) => {
            formError.value[error.path] = error.message;
          });
        } else {
          // Si es un error de Firebase
          switch (err.code) {
            case 'auth/user-not-found':
              formError.value.general = "No user found with this email.";
              break;
            case 'auth/wrong-password':
              formError.value.general = "Incorrect password.";
              break;
            case 'auth/invalid-email':
              formError.value.general = "Invalid email format.";
              break;
            case 'auth/too-many-requests':
              formError.value.general = "Too many login attempts. Please try again later.";
              break;
            default:
              formError.value.general = "Failed to log in. Please check your credentials.";
          }
          console.error("Firebase Auth Error:", err);
        }
      }

      isLoading.value = false;
    };

    return {
      formData,
      onLogin,
      formError,
      isLoading,
    };
  },
};
</script>

<style lang="scss" scoped>
.login {
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

    .error-message {
      color: red;
      font-size: 0.875rem;
      margin-top: 5px;
    }
  }

  .form-switch {
    text-align: center;
    margin-top: 30px;
    cursor: pointer;

    &:hover {
      opacity: 0.6;
    }
  }

  button.loading {
    background-color: #ccc;
    pointer-events: none;
  }
}
</style>
