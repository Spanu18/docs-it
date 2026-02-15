<script setup lang="ts">
import PageHero from '@theme/components/PageHero.vue'

defineProps<{ title?: string }>()
</script>

<template>
  <PageHero>
    <template #title>{{ title || 'Partner di Vue' }}</template>
    <template #lead>
      <p>
        I partner di Vue sono aziende approvate dal team Vue che forniscono
        servizi di prima classe, consulenza e sviluppo Vue. Se la tua azienda
        è interessata ad essere elencata come partner, per favore
        <a class="link" href="https://airtable.com/shrCQhat57SApJI2l" target="_blank"
          >segnala il tuo interesse qui</a
        >.
      </p>
    </template>
  </PageHero>
</template>
