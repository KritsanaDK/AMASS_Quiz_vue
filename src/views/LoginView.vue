<template>
  <main class="auth-page">
    <section class="auth-card">
      <p class="auth-eyebrow">Vue Auth Demo</p>
      <h1 class="auth-title">Login</h1>
      <p class="auth-subtitle">Sign in to continue to your dashboard.</p>

      <form class="auth-form" @submit.prevent="login">
        <label class="field-label" for="user">Username</label>
        <input
          id="user"
          v-model="user"
          class="field-input"
          placeholder="User"
        />

        <label class="field-label" for="password">Password</label>
        <input
          id="password"
          v-model="password"
          class="field-input"
          type="password"
          placeholder="Password"
        />

        <button class="primary-btn" type="submit">Login</button>
      </form>

      <p class="auth-footer">
        <router-link to="/register">Create new account</router-link>
      </p>
    </section>

    <div v-if="showDialog" class="dialog-overlay" @click.self="closeDialog">
      <section class="dialog-card" role="alertdialog" aria-modal="true">
        <h2 class="dialog-title">
          {{ dialogType === "success" ? "Success" : "Error" }}
        </h2>
        <p class="dialog-message">{{ dialogMessage }}</p>
        <button class="primary-btn" type="button" @click="closeDialog">
          OK
        </button>
      </section>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter();
const API_BASE_URL =
  import.meta.env.VITE_API_BASE_URL || "http://127.0.0.1:3001";

const user = ref("");
const password = ref("");
const showDialog = ref(false);
const dialogMessage = ref("");
const dialogType = ref("error");
const pendingRoute = ref("");

const openDialog = (message, type = "error", nextRoute = "") => {
  dialogMessage.value = message;
  dialogType.value = type;
  pendingRoute.value = nextRoute;
  showDialog.value = true;
};

const closeDialog = () => {
  showDialog.value = false;
  if (pendingRoute.value) {
    router.push(pendingRoute.value);
    pendingRoute.value = "";
  }
};

const login = async () => {
  showDialog.value = false;
  pendingRoute.value = "";

  if (!user.value || !password.value) {
    openDialog("Please enter username and password.");
    return;
  }

  try {
    const response = await axios.post(`${API_BASE_URL}/api/v1/auth/login`, {
      username: user.value,
      password: password.value,
    });

    console.log("Login response:", response?.data?.data?.token);

    if (response?.data?.data?.token) {
      localStorage.setItem("token", response.data.data.token);
    }

    openDialog("Login success.", "success", "/users");
  } catch (error) {
    console.error(error);
    openDialog("Invalid username or password.");
  }
};
</script>

<style scoped>
.auth-page {
  min-height: calc(100vh - 4rem);
  display: grid;
  place-items: center;
  padding: 1rem;
}

.auth-card {
  width: min(460px, 100%);
  background: var(--color-background-soft);
  border: 1px solid var(--color-border);
  border-radius: 16px;
  padding: 2rem;
}

.auth-eyebrow {
  color: var(--color-text);
  opacity: 0.75;
  font-size: 0.85rem;
  margin-bottom: 0.25rem;
}

.auth-title {
  color: var(--color-heading);
  font-size: 1.7rem;
  font-weight: 700;
  margin-bottom: 0.25rem;
}

.auth-subtitle {
  margin-bottom: 1.5rem;
  color: var(--color-text);
  opacity: 0.8;
}

.auth-form {
  display: grid;
  gap: 0.85rem;
}

.field-label {
  font-size: 0.9rem;
  color: var(--color-heading);
  font-weight: 600;
}

.field-input {
  width: 100%;
  border-radius: 10px;
  border: 1px solid var(--color-border);
  background: var(--color-background);
  color: var(--color-text);
  padding: 0.7rem 0.85rem;
  outline: none;
}

.field-input:focus {
  border-color: var(--color-border-hover);
}

.primary-btn {
  margin-top: 0.5rem;
  border: 1px solid var(--color-border-hover);
  border-radius: 10px;
  padding: 0.75rem 1rem;
  font-weight: 600;
  cursor: pointer;
  background: var(--color-background-mute);
  color: var(--color-heading);
}

.primary-btn:hover {
  background: var(--color-background);
}

.auth-footer {
  margin-top: 1rem;
  text-align: center;
}

.dialog-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  display: grid;
  place-items: center;
  padding: 1rem;
}

.dialog-card {
  width: min(360px, 100%);
  background: var(--color-background-soft);
  border: 1px solid var(--color-border);
  border-radius: 14px;
  padding: 1.2rem;
  display: grid;
  gap: 0.8rem;
}

.dialog-title {
  color: var(--color-heading);
  font-size: 1.2rem;
  font-weight: 700;
}

.dialog-message {
  color: var(--color-text);
}
</style>
