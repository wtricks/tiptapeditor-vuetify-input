<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

interface Item {
  id: number;
  title: string;
  type: string;
}

const props = defineProps<{
  items: Item[];
  command: (selectedItem: { id: number; label: string }) => void;
}>();

const selectedId = ref<number | null>(null);

const groupedItems = computed(() => {
  const grouped: Record<string, Item[]> = {};
  props.items.forEach(item => {
    if (!grouped[item.type]) {
      grouped[item.type] = [];
    }
    grouped[item.type].push(item);
  });
  return grouped;
});

const hasItems = computed(() => Object.keys(groupedItems.value).length > 0);

function selectItem(item: Item) {
  props.command({ id: item.id, label: item.title });
  selectedId.value = item.id;
}

let keydownHandler: (event: KeyboardEvent) => boolean;

onMounted(() => {
  keydownHandler = (event) => {
    if (event.key === 'ArrowUp') {
      upHandler();
      return true;
    }
    if (event.key === 'ArrowDown') {
      downHandler();
      return true;
    }
    if (event.key === 'Enter') {
      enterHandler();
      return true;
    }
    return false;
  };
  document.addEventListener('keydown', keydownHandler);
});

onBeforeUnmount(() => {
  if (keydownHandler) {
    document.removeEventListener('keydown', keydownHandler);
  }
});

function upHandler() {
  const currentIndex = getCurrentIndex();
  const previousIndex = (currentIndex - 1 + props.items.length) % props.items.length;
  selectedId.value = props.items[previousIndex].id;
}

function downHandler() {
  const currentIndex = getCurrentIndex();
  const nextIndex = (currentIndex + 1) % props.items.length;
  selectedId.value = props.items[nextIndex].id;
}

function enterHandler() {
  const selectedItem = props.items.find(item => item.id === selectedId.value);
  if (selectedItem) {
    selectItem(selectedItem);
  }
}

function getCurrentIndex() {
  return props.items.findIndex(item => item.id === selectedId.value);
}
</script>

<template>
  <div class="items">
    <template v-if="hasItems">
      <div
          v-for="(group, type) in groupedItems"
          :key="type"
          style="margin-bottom: 12px"
      >
        <div class="type-item">
          {{ type }}
        </div>
        <button
            v-for="item in group"
            :key="item.id"
            class="item"
            :class="{ 'is-selected': item.id === selectedId }"
            @click="selectItem(item)"
        >
          - {{ item.title }}
        </button>
      </div>
    </template>
    <div v-else class="item">No result</div>
  </div>
</template>


<style scoped lang="scss">
.items {
  padding: 0.2rem;
  position: relative;
  border-radius: 0.5rem;
  background: #FFF;
  color: rgba(0, 0, 0, 0.8);
  overflow: hidden;
  font-size: 0.9rem;
  box-shadow:
      0 0 0 1px rgba(0, 0, 0, 0.05),
      0px 10px 20px rgba(0, 0, 0, 0.1);
}

.item {
  display: block;
  margin: 0;
  width: 100%;
  text-align: left;
  background: transparent;
  border-radius: 0.4rem;
  border: 1px solid transparent;
  padding: 0.2rem 0.4rem;
}

.type-item {
  display: block;
  padding: 0.2rem 0.4rem;
  font-weight: bold;
}

.item.is-selected {
  border-color: #000;
}
</style>
