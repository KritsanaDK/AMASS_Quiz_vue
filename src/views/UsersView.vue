<template>
  <main class="users-page">
    <section class="users-card">
      <header class="users-header">
        <div>
          <p class="eyebrow">Dashboard</p>
          <h1>Users</h1>
          <p v-if="currentUserName" class="current-user">
            Logged in as: {{ currentUserName }}
          </p>
        </div>
        <button class="logout-link" type="button" @click="logout">
          Logout
        </button>
      </header>

      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>User</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="loading">
              <td class="empty">Loading users...</td>
            </tr>
            <tr v-else-if="errorMessage">
              <td class="empty">{{ errorMessage }}</td>
            </tr>
            <tr v-else-if="users.length === 0">
              <td class="empty">No users found</td>
            </tr>
            <tr v-for="u in users" :key="u.id ?? u.username ?? u.user">
              <td>{{ u.username ?? u.user ?? "-" }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </main>
</template>

<script setup>
import { computed, ref, onMounted } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter();
const API_BASE_URL =
  import.meta.env.VITE_API_BASE_URL || "http://127.0.0.1:3001";

const users = ref([]);
const loading = ref(false);
const errorMessage = ref("");
const currentUser = ref(null);

const currentUserName = computed(
  () => currentUser.value?.username ?? currentUser.value?.user ?? "",
);

const logout = () => {
  localStorage.removeItem("token");
  localStorage.removeItem("currentUser");
  router.push("/");
};

const loadUsers = async () => {
  loading.value = true;
  errorMessage.value = "";

  const token = localStorage.getItem("token");
  if (!token) {
    loading.value = false;
    errorMessage.value = "Please login first.";
    router.push("/");
    return;
  }

  try {
    const response = await axios.get(`${API_BASE_URL}/api/v1/users/me`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    const payload = response.data?.data;

    if (Array.isArray(payload)) {
      users.value = payload;
    } else if (payload && typeof payload === "object") {
      users.value = [payload];
    } else {
      users.value = [];
    }
  } catch (error) {
    console.error(error);
    errorMessage.value = "Cannot load users. Please login again.";
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  const savedUser = localStorage.getItem("currentUser");
  currentUser.value = savedUser ? JSON.parse(savedUser) : null;
  loadUsers();
});
</script>

<style scoped>
.users-page {
  min-height: calc(100vh - 4rem);
  display: grid;
  place-items: center;
  padding: 1rem;
}

.users-card {
  width: min(720px, 100%);
  background: var(--color-background-soft);
  border: 1px solid var(--color-border);
  border-radius: 16px;
  padding: 1.5rem;
}

.users-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1rem;
}

.users-header h1 {
  color: var(--color-heading);
  font-size: 1.5rem;
  font-weight: 700;
}

.eyebrow {
  font-size: 0.85rem;
  opacity: 0.75;
}

.current-user {
  margin-top: 0.2rem;
  opacity: 0.85;
}

.logout-link {
  border: 1px solid var(--color-border-hover);
  border-radius: 10px;
  padding: 0.45rem 0.75rem;
  background: var(--color-background);
  color: var(--color-heading);
  cursor: pointer;
}

.table-wrap {
  border: 1px solid var(--color-border);
  border-radius: 12px;
  overflow: hidden;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  text-align: left;
  padding: 0.75rem 0.9rem;
  border-bottom: 1px solid var(--color-border);
}

th {
  background: var(--color-background-mute);
  color: var(--color-heading);
  font-weight: 700;
}

tbody tr:last-child td {
  border-bottom: none;
}

.empty {
  text-align: center;
  opacity: 0.8;
}
</style>
