<template>
  <section class="search">
    <div class="search__box" :class="{ focused: isFocused }">
      <span class="search__icon">🔎</span>
      <input
        ref="inputEl"
        :value="modelValue"
        :placeholder="placeholder"
        autofocus
        autocomplete="off"
        autocorrect="off"
        autocapitalize="off"
        spellcheck="false"
        @input="
          $emit('update:modelValue', ($event.target as HTMLInputElement).value)
        "
        @focus="isFocused = true"
        @blur="isFocused = false"
        @keyup.enter="handleSearch"
        @touchstart="handleTouchStart"
        @touchend="handleTouchEnd" />
      <button
        v-if="modelValue"
        class="btn btn--ghost"
        type="button"
        @click="
          $emit('update:modelValue', '');
          $emit('reset');
        "
        @touchstart="handleTouchStart"
        @touchend="handleTouchEnd">
        重置
      </button>
      <button
        class="btn btn--primary"
        type="button"
        :disabled="!modelValue || loading"
        @click="handleSearch"
        @touchstart="handleTouchStart"
        @touchend="handleTouchEnd">
        {{ loading ? "搜索中…" : "搜索" }}
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
const props = defineProps<{
  modelValue: string;
  loading: boolean;
  placeholder: string;
}>();
const emit = defineEmits(["update:modelValue", "search", "reset"]);

const isFocused = ref(false);
const inputEl = ref<HTMLInputElement | null>(null);
const touchStartTime = ref(0);

// 處理搜索按鈕點選
function handleSearch() {
  // iOS Safari相容性：確保輸入框失去焦點
  if (
    typeof window !== "undefined" &&
    document.activeElement instanceof HTMLInputElement
  ) {
    document.activeElement.blur();
  }

  // 新增小延遲確保焦點處理完成
  setTimeout(() => {
    emit("search");
  }, 50);
}

// 處理觸控開始事件
function handleTouchStart() {
  touchStartTime.value = Date.now();
}

// 處理觸控結束事件
function handleTouchEnd() {
  const touchDuration = Date.now() - touchStartTime.value;
  // 如果觸控時間太短，可能是誤觸，不執行操作
  if (touchDuration < 50) {
    return;
  }
}

onMounted(() => {
  // 等待一幀后再聚焦，避免與 SSR/過渡階段衝突
  requestAnimationFrame(() => {
    // iOS Safari相容性：使用setTimeout確保DOM完全渲染
    setTimeout(() => {
      inputEl.value?.focus();
    }, 100);
  });
});
</script>

<style scoped>
.search {
  margin-top: 16px;
}
.search__box {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 12px;
  border-radius: 14px;
  border: 1px solid #e5e7eb;
  background: #fff;
  box-shadow: 0 2px 16px rgba(0, 0, 0, 0.04);
  /* iOS Safari相容性：防止縮放 */
  -webkit-text-size-adjust: 100%;
  -webkit-tap-highlight-color: transparent;
}
.search__box.focused {
  box-shadow: 0 10px 30px rgba(38, 132, 255, 0.12);
}
.search__icon {
  opacity: 0.6;
  /* iOS Safari相容性：防止圖示被縮放 */
  -webkit-user-select: none;
  user-select: none;
}
.search__box input {
  flex: 1;
  border: 0;
  outline: none;
  font-size: 16px;
  /* iOS Safari相容性：防止輸入框縮放 */
  -webkit-appearance: none;
  -webkit-border-radius: 0;
  border-radius: 0;
  /* iOS Safari相容性：防止自動縮放 */
  -webkit-text-size-adjust: 100%;
  /* iOS Safari相容性：改善輸入體驗 */
  -webkit-tap-highlight-color: transparent;
}
.btn {
  padding: 8px 12px;
  border: 1px solid #e5e7eb;
  background: #fff;
  color: #111;
  border-radius: 10px;
  cursor: pointer;
  /* iOS Safari相容性：防止按鈕縮放 */
  -webkit-appearance: none;
  -webkit-tap-highlight-color: transparent;
  /* iOS Safari相容性：改善觸控體驗 */
  -webkit-user-select: none;
  user-select: none;
  /* iOS Safari相容性：防止按鈕被縮放 */
  -webkit-transform: translateZ(0);
  transform: translateZ(0);
}
.btn:hover {
  background: #f6f7f9;
}
.btn:active {
  /* iOS Safari相容性：觸控反饋 */
  background: #e5e7eb;
  transform: scale(0.98);
}
.btn[disabled] {
  opacity: 0.6;
  cursor: not-allowed;
  pointer-events: none;
}

.btn--primary {
  background: #111;
  color: #fff;
  border-color: #111;
}
.btn--primary:hover {
  background: #000;
}
.btn--primary:active {
  background: #000;
}
.btn--ghost {
  background: transparent;
}

/* 小屏優化：按鈕換行、輸入佔滿 */
@media (max-width: 640px) {
  .search__box {
    flex-wrap: wrap;
    gap: 6px;
  }
  .search__icon {
    display: none;
  }
  .search__box input {
    width: 100%;
    font-size: 15px;
    /* iOS Safari相容性：確保輸入框在小螢幕上正常工作 */
    -webkit-appearance: none;
    -webkit-border-radius: 0;
  }
  .btn {
    padding: 8px 10px;
    font-size: 14px;
    /* iOS Safari相容性：確保按鈕在小螢幕上正常工作 */
    min-height: 44px;
    min-width: 44px;
  }
}

/* iOS Safari特定優化 */
@supports (-webkit-touch-callout: none) {
  .search__box input {
    /* iOS Safari相容性：防止輸入框出現預設樣式 */
    -webkit-appearance: none;
    -webkit-border-radius: 0;
    border-radius: 0;
  }

  .btn {
    /* iOS Safari相容性：確保按鈕有足夠的觸控區域 */
    min-height: 44px;
    min-width: 44px;
    /* iOS Safari相容性：防止按鈕出現預設樣式 */
    -webkit-appearance: none;
    -webkit-border-radius: 10px;
  }
}
</style>
