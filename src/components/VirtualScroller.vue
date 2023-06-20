<template>
    <div :class="$style['virtual-scroller']" ref="containerRef" @scroll="onScroll">
        <div :class="$style['virtual-scroller__container']">
            <template v-for="(item, index) in limitedItems" :key="index">
                <slot :item="item" />
            </template>
        </div>
    </div>
</template>

<script setup lang="ts" generic="T">
import { ref, type Ref, shallowRef, computed, watch, onMounted } from 'vue';

interface Props {
    items: T[];
    itemHeight?: number | string;
}

const props = defineProps<Props>();


const containerRef = ref() as Ref<HTMLDivElement>;

const DIRECTION_UP = -1;
const DIRECTION_DOWN = 1;

const first = shallowRef(0);

const baseItemHeight = shallowRef(props.itemHeight);

const innerItemHeight = computed({
    get() {
        return parseInt((baseItemHeight.value ?? 0) as string, 10);
    },
    set(value) {
        baseItemHeight.value = value;
    },
});

// массив изначальных размеров элементов
const itemSizes = computed(() => {
    const sizes: number[] = [];

    for (let i = 0; i < props.items.length; i++) {
        sizes.push(innerItemHeight.value);
    }

    return sizes;
});

const MAXIMUM_VISIBLE_ITEMS = 12;

const visibleItems = computed(() => {
    const height = containerRef.value?.getBoundingClientRect()?.height;
    if (!height) {
        return 0;
    }
    return innerItemHeight.value ? Math.max(MAXIMUM_VISIBLE_ITEMS, Math.ceil((height / innerItemHeight.value) * 1.7 + 1)) : MAXIMUM_VISIBLE_ITEMS;
});

function calculateOffset(index: number) {
    return itemSizes.value.slice(0, index)
        .reduce((acc, size) => {
            return acc + (size || innerItemHeight.value)
        }, 0);
}

function calculateMiddleIndex(scrollTop: number) {
    const length = props.items.length;
    let middleIndex = 0;
    let middleOffset = 0;
    while (middleOffset < scrollTop && middleIndex < length) {
        middleOffset += itemSizes.value[middleIndex++] || innerItemHeight.value;
    }

    return middleIndex - 1;
}

const last = computed(() => {
    return Math.min(props.items.length, first.value + visibleItems.value);
});

const limitedItems = computed(() => {
    return props.items.slice(first.value, last.value);
});

const paddingTop = computed(() => {
    return `${calculateOffset(first.value)}px`;
});
const paddingBottom = computed(() => {
    const all = calculateOffset(props.items.length);
    const bottom = all - calculateOffset(last.value);
    return `${bottom}px`;
});

onMounted(() => {
   
    if (!innerItemHeight.value) {
        const sumSizes = itemSizes.value.slice(first.value, last.value).reduce((acc, size) => {
            return acc + size;
        }, 0);

        innerItemHeight.value = sumSizes / visibleItems.value;
    }
});

// watch(() => props.items.length, () => {

// });

let lastScrollTop = 0;

function onScroll() {
    if (!containerRef.value) {
        return;
    }

    const height = containerRef.value.getBoundingClientRect().height;
    
    const scrollTop = containerRef.value.scrollTop;

    const direction = scrollTop < lastScrollTop ? DIRECTION_UP : DIRECTION_DOWN;

    const middleIndex = calculateMiddleIndex(scrollTop + height / 2);

    const buffer = Math.round(visibleItems.value / 3);

    const firstIndex = middleIndex - buffer;
    const lastIndex = first.value + (buffer * 2) - 1;

    if (direction === DIRECTION_UP && middleIndex <= lastIndex) {
        first.value = Math.max(0, Math.min(props.items.length, firstIndex));
    } else if (direction === DIRECTION_DOWN && middleIndex >= lastIndex) {
        first.value = Math.max(0, Math.min(props.items.length - visibleItems.value, firstIndex));
    }

    lastScrollTop = scrollTop;
}

</script>

<style module>

.virtual-scroller {
    display: block;
    flex: 1 1 auto;
    max-width: 100%;
    overflow: auto;
    position: relative;
}

.virtual-scroller__container {
    display: block;
    padding-top: v-bind('paddingTop');
    padding-bottom: v-bind('paddingBottom');
}

</style>