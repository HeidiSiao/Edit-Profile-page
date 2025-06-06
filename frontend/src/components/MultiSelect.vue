<script setup>
// 父傳入使用者選中的選項: 你現在要顯示這些選項 props 唯讀不能被改寫
const props = defineProps({
  modelValue: {
    type: [Array, String, Number], // 支援多選陣列或單選值
    default: () => [],
  },
  options: {
    type: Array,
    required: true,
  },
  labelKey: String, // 畫面上的文字
  valueKey: String, //指定取值屬性的變數，傳遞或儲存的資料
  cols: {
    type: [Number, String],
    default: 4,
  },
  multiple: {
    type: Boolean,
    default: true,
  },
});
const emit = defineEmits(["update:modelValue"]);

// 取得選項值 選項是物件
const getOptionValue = (option) => {
  return props.valueKey && typeof option === "object"
    ? option[props.valueKey]
    : option;
};
// 選項是基本型別
const optionLabel = (option) => {
  return props.labelKey && typeof option === "object"
    ? option[props.labelKey] //只有物件才會有屬性可以用
    : option;
};

const optionKey = (option) => {
  return getOptionValue(option);
};

// 先 getOptionValue 取得選項「值」
// 多選：判斷 modelValue 是陣列才用 includes，避免錯誤
// 單選：直接判斷相等就是被選中
const isSelected = (option) => {
  const val = getOptionValue(option);
  if (props.multiple) {
    if (!Array.isArray(props.modelValue)) return false; // 防呆
    return props.modelValue.includes(val);
  } else {
    return props.modelValue === val;
  }
};

// 根據多選、單選，加進陣列 或 直接取代
const toggle = (option) => {
  const val = getOptionValue(option);
  // 如果是陣列
  if (props.multiple) {
    const newValue = Array.isArray(props.modelValue)
      ? [...props.modelValue]
      : [];
    const index = newValue.indexOf(val);
    if (index === -1) {
      newValue.push(val);
    } else {
      newValue.splice(index, 1);
    }
    emit("update:modelValue", newValue);
  } else {
    // 單選：如果點同一個就取消，否則直接切換
    const newValue = props.modelValue === val ? null : val;
    emit("update:modelValue", newValue);
  }
};
</script>

<!-- 支援多選功能 -->
<template>
  <!-- 動態決定 grid 欄數 -->
  <div :class="`grid gap-4 px-4 py-2 grid-cols-${cols}`">
    <!-- 產生多個標籤按鈕 -->
    <button
      v-for="option in options"
      :key="optionKey(option)"
      @click="toggle(option)"
      class="px-2 py-1 text-sm font-semibold transition border-2 rounded-lg"
      :class="
        isSelected(option)
          ? 'bg-green-100 border-green-500 text-green-600'
          : 'border-gray-300 text-gray-500'
      "
    >
      {{ optionLabel(option) }}
    </button>
  </div>
</template>
