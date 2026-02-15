<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <p>Componente corrente: A</p>
  <span style="margin-right: 20px">Conta: {{ count }}</span>
  <button @click="count++">+</button>
</template>
