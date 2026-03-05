<template>
  <main class="auth-page">
    <section class="auth-card">
      <p class="auth-eyebrow">Vue Auth Demo</p>
      <h1 class="auth-title">Register</h1>
      <p class="auth-subtitle">Create your account in a few seconds.</p>

      <p v-if="notice" class="notice" :class="`notice-${noticeType}`">
        {{ notice }}
      </p>

      <form class="auth-form" @submit.prevent="register">
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

        <label class="field-label" for="confirm">Confirm Password</label>
        <input
          id="confirm"
          v-model="confirm"
          class="field-input"
          type="password"
          placeholder="Confirm Password"
        />

        <button class="primary-btn" type="submit">Register</button>
      </form>

      <p class="auth-footer">
        <router-link to="/">Back to Login</router-link>
      </p>
    </section>
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
const confirm = ref("");
const notice = ref("");
const noticeType = ref("error");

const setNotice = (message, type = "error") => {
  notice.value = message;
  noticeType.value = type;
};

const register = async () => {
  notice.value = "";

  if (password.value !== confirm.value) {
    setNotice("Password does not match.");
    return;
  }

  if (!user.value || !password.value) {
    setNotice("Please enter username and password.");
    return;
  }

  try {
    await axios.post(`${API_BASE_URL}/api/v1/auth/register`, {
      username: user.value,
      password: password.value,
    });

    setNotice("Register success.", "success");
    setTimeout(() => router.push("/"), 700);
  } catch (error) {
    console.error(error);
    setNotice("Register failed. Please try again.");
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

.notice {
  margin-bottom: 1rem;
  border-radius: 10px;
  padding: 0.65rem 0.8rem;
  font-size: 0.92rem;
  border: 1px solid var(--color-border-hover);
}

.notice-error {
  background: var(--color-background-mute);
  color: var(--color-heading);
}

.notice-success {
  background: var(--color-background);
  color: var(--color-heading);
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
</style>
