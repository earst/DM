<script  setup>
import { useCounterStore } from '@/stores/counter';
import { computed } from 'vue'
const counter = useCounterStore()

const count = computed(() => counter.count)
const doubleCount = computed(() => counter.doubleCount)
const reduce =()=>{
  counter.decrement()
}

const add =()=>{
  counter.increment()
}
</script>

<template>
  <div>用户列表</div>
  <button @click="reduce">-10</button>
  {{ count }} - {{ counter.count }}  - {{ counter.doubleCount }} -{{  doubleCount }}
  <button @click="add">+10</button>
</template>