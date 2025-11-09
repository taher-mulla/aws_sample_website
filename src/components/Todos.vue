<script setup lang="ts">
import "@/assets/main.css";
import { ref } from "vue";

const API_URL = "https://8j8uwfup9c.execute-api.us-east-1.amazonaws.com/default/add";

const num1 = ref<string>("");
const num2 = ref<string>("");
const loading = ref(false);
const result = ref<number | null>(null);
const error = ref<string | null>(null);

async function submitForm() {
  error.value = null;
  result.value = null;

  const first = Number(num1.value);
  const second = Number(num2.value);

  if (isNaN(first) || isNaN(second)) {
    error.value = "Please enter valid numbers.";
    return;
  }

  loading.value = true;
  try {
    const response = await fetch(API_URL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        num1: first,
        num2: second,
      }),
    });

    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }

    const data = (await response.json()) as unknown;
    console.log("data from API", data);

    if (
      typeof data !== "object" ||
      data === null ||
      !("result" in data) ||
      typeof (data as { result: unknown }).result !== "number"
    ) {
      throw new Error("Unexpected response from server.");
    }

    result.value = (data as { result: number }).result;
  } catch (err) {
    error.value = err instanceof Error ? err.message : "Something went wrong.";
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <main class="addition-form">
    <h1>Number Adder</h1>
    <form @submit.prevent="submitForm">
      <label>
        Number 1
        <input v-model="num1" type="number" required />
      </label>
      <label>
        Number 2
        <input v-model="num2" type="number" required />
      </label>
      <button type="submit" :disabled="loading">
        {{ loading ? "Calculating..." : "Add Numbers" }}
      </button>
    </form>

    <p v-if="result !== null" class="result">Result: {{ result }}</p>
    <p v-if="error" class="error">{{ error }}</p>
  </main>
</template>
