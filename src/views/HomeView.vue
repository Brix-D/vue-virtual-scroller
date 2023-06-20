<script setup lang="ts">

</script>

<template>
  <main :class="$style['main']">
    <ul :class="$style['list']">
      <li :class="$style['list-item']" v-for="i in 100000" :key="i">
        list item #{{ i }}
      </li>
    </ul>
  </main>
</template>

<style module>
.main {
  padding: 20px;
}
.list {
  list-style-type: none;
  padding: 0;
}

.list-item {
  height: 56px;
  background-color: #fff;
  border: 1px solid #08ad44;
  border-radius: 8px;
  padding: 16px;
}
</style>