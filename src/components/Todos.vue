<script setup lang="ts">
import "@/assets/main.css";
import { fetchAuthSession } from "aws-amplify/auth";
import { ref } from "vue";

const API_URL = "https://8j8uwfup9c.execute-api.us-east-1.amazonaws.com/default/add_user";

const nameInput = ref<string>("");
const loading = ref(false);
const message = ref<string | null>(null);
const error = ref<string | null>(null);

async function submitForm() {
  error.value = null;
  message.value = null;

  const trimmedName = nameInput.value.trim();
  if (!trimmedName) {
    error.value = "Please enter a name.";
    return;
  }

  loading.value = true;
  try {
    const { tokens } = await fetchAuthSession();
    const idToken = tokens?.idToken?.toString();

    if (!idToken) {
      throw new Error("Auth session missing ID token.");
    }

    const response = await fetch(API_URL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${idToken}`,
      },
      body: JSON.stringify({
        name: trimmedName,
      }),
    });

    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }

    const data = (await response.json()) as unknown;

    if (typeof data !== "string") {
      throw new Error("Unexpected response from server.");
    }

    message.value = data;
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Something went wrong.";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <main class="addition-form">
    <h1>Say Hello</h1>
    <form @submit.prevent="submitForm">
      <label>
        Name
        <input v-model="nameInput" type="text" required />
      </label>
      <button type="submit" :disabled="loading">
        {{ loading ? "Sending..." : "Submit" }}
      </button>
    </form>

    <p v-if="message" class="result">{{ message }}</p>
    <p v-if="error" class="error">{{ error }}</p>
  </main>
</template>
