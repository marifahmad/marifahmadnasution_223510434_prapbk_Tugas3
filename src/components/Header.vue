<template>
    <div>
        <h1 style="background-color:#fff; padding: 10px;">
            {{ title }}
        </h1>
    </div>
    </template>

<script setup>
import { ref, defineProps } from 'vue'

const title =ref('Post And To do List')
const props = defineProps({
  title: String,
})
</script>