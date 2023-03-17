<script setup>
import { ref } from "vue";
import NavbarDrawer from "./NavbarDrawer.vue";

const toggleLeftDrawer = ref(false);
</script>

<template>
  <q-header elevated>
    <q-toolbar>
      <q-btn
        flat
        dense
        round
        icon="menu"
        aria-label="Menu"
        @click="toggleLeftDrawer = !toggleLeftDrawer"
      />

      <q-toolbar-title> Prueba Super-Heros </q-toolbar-title>

    </q-toolbar>
  </q-header>

  <q-drawer
    v-model="toggleLeftDrawer"
    show-if-above
    :width="270"
  >
    <NavbarDrawer />
  </q-drawer>
</template>
