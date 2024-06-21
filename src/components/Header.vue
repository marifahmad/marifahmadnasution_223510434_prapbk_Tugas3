<template>
    <div>
        <h1 style=" background:transparent;
    border: 2px solid rgb(255, 255, 255, .2);
    border-radius: 20px;
    backdrop-filter: blur(20px);
    box-shadow: 0 0 10px rgba(0, 0, 0, .2);; padding: 10px;">
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