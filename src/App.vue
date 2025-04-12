<script setup lang="ts">
import { ref, computed, watch, nextTick, type Ref } from 'vue'
import { normalize } from '@/utils/normalize'

const names: string[] = [
  "Alice", "Bob", "Alex", "John", "Michael", "Emma", "Olivia", "Liam",
  "Sophia", "James", "Daniel", "Алина", "Максим", "Сергей", "Мария",
  "Ольга", "Алексей", "Ирина", "Ирина2", "ира маленькая", "ирина", "Ирина3", "Иришка", "Ирунчик",
  "Татьяна", "Никита", "Владимир", "Екатерина", "Жанна", "Роман", "Антон", "Юлия", "Светлана",
  "Дмитрий", "Валентина", "Евгений", "Кристина", "Людмила", "Григорий", "Степан"
]

const searchText: Ref<string> = ref('')
const debouncedText: Ref<string> = ref('')
const selectedIndex: Ref<number> = ref(-1)
const debounceTimeout: Ref<ReturnType<typeof setTimeout> | null> = ref(null)
const isFocus: Ref<boolean> = ref(false)


const listRef = ref<HTMLElement | null>(null)

watch(selectedIndex, (newIndex) => {
  nextTick(() => {
    const list = listRef.value
    const item = list?.children[newIndex] as HTMLElement | undefined
    if (item && list) {
      const offset = item.offsetTop - list.offsetTop
      list.scrollTop = offset - 20
    }
  })
})

const onInput = (): void => {
  if (debounceTimeout.value) {
    clearTimeout(debounceTimeout.value)
  }
  debounceTimeout.value = setTimeout(() => {
    debouncedText.value = searchText.value.trim()
    isFocus.value = true
  }, 300)
}

const filteredNames = computed<string[]>(() => {
  const query = normalize(debouncedText.value)
  if (!query) return []
  return names.filter(name => normalize(name).includes(query))
})

const showSuggestions = computed<boolean>(() => debouncedText.value.length > 0)

const selectName = (name: string): void => {
  searchText.value = name
  debouncedText.value = name
  selectedIndex.value = -1
  isFocus.value = false
}

const keyDown = (): void => {
  if (filteredNames.value.length === 0) return
  selectedIndex.value = (selectedIndex.value + 1) % filteredNames.value.length
}

const keyUp = (): void => {
  if (filteredNames.value.length === 0) return
  selectedIndex.value =
    (selectedIndex.value - 1 + filteredNames.value.length) % filteredNames.value.length
}

const selectHighlighted = (): void => {
  if (selectedIndex.value >= 0) {
    selectName(filteredNames.value[selectedIndex.value])
  }
}

const onBlur = (): void => {
  setTimeout(() => {
    isFocus.value = false
  }, 200)
}
</script>
<template>
  <div class="name-search-box">
    <div class="name-search">
      <div class="name-search__title">Поиск</div>
      <input
        v-model="searchText"
        @input="onInput"
        @keydown.down.prevent="keyDown"
        @keydown.up.prevent="keyUp"
        @keydown.enter.prevent="selectHighlighted"
        @focus="isFocus = true"
        @blur="onBlur"
        type="text"
        placeholder="Введите имя"
        class="name-search__input"
      />
      <transition name="fade">
        <div v-show="showSuggestions && isFocus" class="name-search__dropdown">
          <ul class="name-search__result"
              ref="listRef"
              v-memo="[filteredNames, selectedIndex]">
            <li
              v-for="(name, index) in filteredNames"
              :key="index"
              @click="selectName(name)"
              @mousedown.prevent="selectName(name)"
            >
              <span class="name-search__item" :class="{ highlighted: index === selectedIndex }">{{ name }}</span>
            </li>
            <li v-if="filteredNames.length === 0" class="no-results">
              Ничего не найдено
            </li>
          </ul>
        </div>
      </transition>
    </div>
  </div>
</template>

<style scoped lang="scss">
.name-search {
  width: 100%;
  max-width: 400px;
  margin: 0 auto;
  position: relative;
  &__title {
    font-size: 18px;
    font-weight: bold;
    padding-bottom: 10px;
  }
  &__input {
    width: 100%;
    padding: 8px 12px;
    border: 1px solid #ccc;
    border-radius: 8px;
    font-size: 16px;
  }
  &-box {
    width: 100%;
    max-width: 1000px;
    margin: 0 auto;
    background: #99DDFF;
    padding: 30px;
  }
  &__item {
    display: block;
    padding: 10px;
    cursor: pointer;
    transition: background-color 0.2s;
    &:hover,
    &.highlighted {
      background-color: #4BA1FF;
      color: #fff;
    }
  }
  &__result {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: #fff;
    border: 1px solid #ddd;
    border-radius: 8px;
    margin-top: 4px;
    width: 100%;
    max-height: 200px;
    overflow-y: auto;
    z-index: 1000;
    list-style: none;
    padding: 0;
  }
}

.no-results {
  padding: 10px;
  color: #888;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
