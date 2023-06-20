<template>
  <main :class="$style['main']">
    <div :class="$style['list']">
      <virtual-scroller :items="list" :item-height="56">
        <template #default="{ item }">
          <div :class="$style['list-item']" :key="item">
            list item #{{ item }}
          </div>
        </template>
      </virtual-scroller>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import VirtualScroller from '@/components/VirtualScroller.vue';

const list = computed(() => {
  const list = [];
  for (let i = 1; i <= 100000; i++) {
    list.push(i);
  }
  return list;
});

</script>

<style module>
.main {
  padding: 20px;
  height: 100vh;
  overflow-y: auto;
  overflow-x: hidden;
  position: relative;
  display: flex;
  flex-direction: column;
}
.list {
  list-style-type: none;
  padding: 0;
  height: 0;
  flex-grow: 1;
  display: flex;
}

.list-item {
  height: 56px;
  background-color: #fff;
  border: 1px solid #08ad44;
  border-radius: 8px;
  padding: 16px;
}
</style>