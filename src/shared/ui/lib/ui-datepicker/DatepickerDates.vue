<template>
  <div class="grid grid-cols-7">
    <div v-for="weekday in WEEKDAYS_LIST" :key="weekday" class="flex h-8 items-center justify-center">
      <UiTypo level="5" class="font-medium text-slate-500">{{ weekday }}</UiTypo>
    </div>
    <div v-for="day in prevMonthDays" :key="day" class="flex h-10 items-center justify-center">
      <button
        type="button"
        class="h-8 w-8 rounded-xl opacity-40 disabled:line-through"
        :class="{ 'text-orange-500': isWeekend(day, -1) }"
        :disabled="!checkAvailableDate(day, -1)"
        @click="selectDate(day, -1)"
      >
        <UiTypo class="font-semibold">{{ day }}</UiTypo>
      </button>
    </div>
    <div v-for="day in days" :key="day" class="flex h-10 items-center justify-center">
      <button
        type="button"
        class="disabled: h-8 w-8 rounded-xl disabled:line-through disabled:opacity-40"
        :class="{ 'bg-blue-500 !text-white': isSelectedDate(day), 'text-orange-500': isWeekend(day) }"
        :disabled="!checkAvailableDate(day)"
        @click="selectDate(day)"
      >
        <UiTypo class="font-semibold">{{ day }}</UiTypo>
      </button>
    </div>
    <div v-for="day in nextMonthDays" :key="day" class="flex h-10 items-center justify-center">
      <button
        type="button"
        class="h-8 w-8 rounded-xl opacity-40 disabled:line-through"
        :class="{ 'text-orange-500': isWeekend(day, 1) }"
        :disabled="!checkAvailableDate(day, 1)"
        @click="selectDate(day, 1)"
      >
        <UiTypo class="font-semibold">{{ day }}</UiTypo>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import dayjs from 'dayjs';
import { computed } from 'vue';
import { UiTypo } from '@/shared/ui';
import { WEEKDAYS_LIST } from './const';

const props = defineProps<{
  modelValue?: dayjs.Dayjs;
  visibleDate: dayjs.Dayjs;
  minDate: dayjs.Dayjs;
  maxDate: dayjs.Dayjs;
}>();

const emit = defineEmits<{
  (e: 'update:model-value', value: dayjs.Dayjs): void;
  (e: 'update:visible-date', value: dayjs.Dayjs): void;
}>();

const startOfMonth = computed(() => {
  return props.visibleDate.startOf('month');
});

const daysOffset = computed(() => {
  return startOfMonth.value.weekday();
});

const days = computed(() => {
  return startOfMonth.value.daysInMonth();
});

const nextMonthDays = computed(() => {
  return 42 - days.value - daysOffset.value;
});

const prevMonthDays = computed(() => {
  const count = startOfMonth.value.subtract(1, 'month').daysInMonth();
  return Array.from({ length: daysOffset.value }, (_, i) => count - i).reverse();
});

function checkAvailableDate(day: number, offset: number = 0) {
  const date = startOfMonth.value.add(offset, 'month').date(day);
  return date.isBetween(props.minDate, props.maxDate, 'day', '[]');
}

function isSelectedDate(day: number) {
  if (!props.modelValue) return false;
  return startOfMonth.value.date(day).isSame(props.modelValue, 'day');
}

function isWeekend(day: number, offset: number = 0) {
  const date = startOfMonth.value.add(offset, 'month').date(day);
  return date.day() === 0 || date.day() === 6;
}

function selectDate(day: number, offset: number = 0) {
  const newDate = startOfMonth.value.add(offset, 'month').date(day);
  emit('update:model-value', newDate);
  if (offset !== 0) {
    emit('update:visible-date', newDate);
  }
}
</script>