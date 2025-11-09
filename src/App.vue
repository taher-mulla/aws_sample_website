<script setup lang="ts">
import { Authenticator } from "@aws-amplify/ui-vue";
import "@aws-amplify/ui-vue/styles.css";
import Todos from "./components/Todos.vue";
</script>

<template>
  <Authenticator>
    <template #default="{ signOut }">
      <main class="app-shell">
        <header class="app-header">
          <button type="button" @click="signOut">Sign out</button>
        </header>
        <Todos />
      </main>
    </template>
  </Authenticator>
</template>

