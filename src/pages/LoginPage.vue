<template>
  <q-page class="q-pa-md">
    <div class="login-wrapper">
      <q-form @submit="login" style="width: 500px">
        <div class="text-h6 q-mb-md">Please Login</div>
        <q-input
          v-model="username"
          label="Username"
          :rules="[(val) => !!val || 'Field is required']"
          outlined
        />
        <q-input
          v-model="password"
          label="Password"
          type="password"
          :rules="[(val) => !!val || 'Field is required']"
          outlined
        />
        <div class="full-width flex flex-center">
          <q-btn
            label="Login"
            type="submit"
            color="black"
            class="full-width q-py-md"
          />
        </div>
      </q-form>
    </div>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import { Loading } from "quasar";

const username = ref("");
const password = ref("");
const router = useRouter();

const login = async () => {
  try {
    Loading.show({
      message: "Đang đăng nhập...",
    });
    axios.defaults.headers.common["Access-Control-Allow-Origin"] = "*";

    const response = await axios.post(
      "https://script.google.com/macros/s/AKfycbwLnFuwFg0TwlFxD7z8x6Fx2dYsh-IWoFQeYkzdAwvHhrddhNi5TuEgMRwj1TkAc-Ek/exec",
      {
        action: "login",
        username: username.value,
        password: password.value,
      }
    );

    if (response.data.success) {
      localStorage.setItem("username", username.value);
      localStorage.setItem("password", password.value);
      router.push("/data");
    } else {
      alert("Login failed");
    }

    Loading.hide();
  } catch (error) {
    console.error("Error logging in:", error);
  }
};
</script>

<style lang="scss" scoped>
.login-wrapper {
  height: 80vh;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
