<template>
  <div class="page-container min-h-screen bg-gray-50 transition-colors dark:bg-gray-900">
    <!-- Header -->
    <div class="rounded-b-3xl bg-gradient-to-br from-emerald-500 to-teal-600 px-6 pb-8 pt-10 text-white shadow-lg">
      <div class="mb-4">
        <p class="text-xs font-medium uppercase tracking-wider text-emerald-200">
          {{ $t("statistics.subtitle") }}
        </p>
        <h1 class="mt-0.5 text-xl font-bold text-white">
          {{ $t("statistics.title") }}
        </h1>
      </div>

      <SummaryBar
        :totalExpense="filteredExpense"
        :totalIncome="filteredIncome"
        :balance="filteredBalance"
        labelClass="text-emerald-200"
        valueClass="text-lg"
      />
    </div>

    <div class="mt-5 px-4">
      <!-- Date Filter (default to year) -->
      <DateFilterBar
        v-if="store.personalRecords.length > 0"
        :dates="recordDates"
        initialMode="year"
        hideDateMode
        @change="onFilterChange"
        class="mb-5"
      />

      <!-- Empty State -->
      <div v-if="filteredRecords.length === 0" class="empty-state py-16">
        <div class="mb-3 text-5xl">📊</div>
        <p class="font-bold">{{ $t("statistics.noData") }}</p>
      </div>

      <template v-else>
        <!-- Shared expense / income toggle -->
        <div class="mb-4 flex justify-end">
          <div class="flex gap-1 rounded-xl bg-gray-100 p-1 dark:bg-gray-800">
            <button
              @click="categoryTab = 'expense'"
              :class="[
                'rounded-lg px-3 py-1 text-xs font-semibold transition-colors',
                categoryTab === 'expense'
                  ? 'bg-white text-red-600 shadow-sm dark:bg-gray-700 dark:text-red-400'
                  : 'text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200',
              ]"
            >
              {{ $t("common.expense") }}
            </button>
            <button
              @click="categoryTab = 'income'"
              :class="[
                'rounded-lg px-3 py-1 text-xs font-semibold transition-colors',
                categoryTab === 'income'
                  ? 'bg-white text-green-600 shadow-sm dark:bg-gray-700 dark:text-green-400'
                  : 'text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200',
              ]"
            >
              {{ $t("common.income") }}
            </button>
          </div>
        </div>

        <!-- Trend Bar Chart -->
        <TrendChart
          v-if="dateFilter.mode !== 'date'"
          :title="trendTitle"
          :data="trendData"
          :maxVal="trendMaxVal"
          :activeTab="categoryTab"
        />

        <!-- Category Breakdown -->
        <CategoryBreakdown
          :data="categoryBreakdown"
          :total="categoryTotal"
          :activeTab="categoryTab"
        />
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { useI18n } from "vue-i18n";
import { useTrackerStore } from "../stores/tracker";
import { colorMap } from "../utils/category";
import SummaryBar from "../components/SummaryBar.vue";
import DateFilterBar from "../components/DateFilterBar.vue";
import TrendChart from "../components/statistics/TrendChart.vue";
import CategoryBreakdown from "../components/statistics/CategoryBreakdown.vue";
import type { DateFilter } from "../components/DateFilterBar.vue";

const store = useTrackerStore();
const { te, t } = useI18n();

// ---- Date Filter (default: year) ----
const today = new Date().toISOString().split("T")[0];
const currentYear = today.slice(0, 4);
const currentMonth = today.slice(5, 7);

const dateFilter = ref<DateFilter>({ mode: "year", year: currentYear, month: currentMonth, date: today });
const onFilterChange = (f: DateFilter) => { dateFilter.value = f; };
const recordDates = computed(() => store.personalRecords.map((r) => r.date));

const filteredRecords = computed(() => {
  const records = store.personalRecords;
  const { mode, year, month, date } = dateFilter.value;
  let result;
  if (mode === "all") result = records;
  else if (mode === "year") result = records.filter((r) => r.date.startsWith(year));
  else if (mode === "month") result = records.filter((r) => r.date.startsWith(`${year}-${month}`));
  else if (mode === "date") result = records.filter((r) => r.date === date);
  else result = records;
  return result;
});

const filteredExpense = computed(() =>
  filteredRecords.value.filter((r) => r.type === "expense").reduce((s, r) => s + r.amount, 0),
);
const filteredIncome = computed(() =>
  filteredRecords.value.filter((r) => r.type === "income").reduce((s, r) => s + r.amount, 0),
);
const filteredBalance = computed(() => filteredIncome.value - filteredExpense.value);

// ---- Shared toggle for trend + category ----
const categoryTab = ref<"expense" | "income">("expense");

// ---- Category Breakdown ----
const categoryMap = computed(() =>
  new Map(store.allCategories.map((c) => [c.name, c])),
);

const getCategoryStyle = (categoryName: string) => {
  const cat = categoryMap.value.get(categoryName);
  const color = cat?.color ?? "gray";
  return {
    icon: cat?.icon ?? "more_horiz",
    ...(colorMap[color] ?? colorMap.gray),
  };
};

const getLocalizedCategoryName = (categoryName: string) => {
  const cat = categoryMap.value.get(categoryName);
  if (cat && te(`categories.${cat.id}`)) return t(`categories.${cat.id}`);
  return categoryName;
};

const categoryTotal = computed(() => {
  return filteredRecords.value
    .filter((r) => r.type === categoryTab.value)
    .reduce((s, r) => s + r.amount, 0);
});

const categoryBreakdown = computed(() => {
  const typeRecords = filteredRecords.value.filter((r) => r.type === categoryTab.value);
  const total = typeRecords.reduce((s, r) => s + r.amount, 0);
  if (total === 0) return [];

  const map = new Map<string, { total: number; count: number }>();
  for (const r of typeRecords) {
    const entry = map.get(r.category) ?? { total: 0, count: 0 };
    entry.total += r.amount;
    entry.count += 1;
    map.set(r.category, entry);
  }

  return [...map.entries()]
    .map(([categoryName, data]) => ({
      categoryName,
      localizedName: getLocalizedCategoryName(categoryName),
      total: Math.round(data.total),
      count: data.count,
      percentage: Math.round((data.total / total) * 100),
      style: getCategoryStyle(categoryName),
    }))
    .sort((a, b) => b.total - a.total);
});

// ---- Dynamic Trend ----
const trendTitle = computed(() => {
  const mode = dateFilter.value.mode;
  if (mode === "all") return t("statistics.yearlyTrend");
  if (mode === "year") return t("statistics.monthlyTrend");
  if (mode === "month") return t("statistics.dailyTrend");
  return "";
});

// Max value for Y-axis (reactive via ref updated inside computed)
const trendMaxValRaw = ref(0);
const trendMaxVal = computed(() => trendMaxValRaw.value);

const trendData = computed(() => {
  const records = filteredRecords.value;
  const mode = dateFilter.value.mode;
  const type = categoryTab.value;
  if (records.length === 0 || mode === "date") return [];

  // Only use records matching current toggle
  const typeRecords = records.filter((r) => r.type === type);

  const getKey = (dateStr: string): string => {
    if (mode === "all") return dateStr.slice(0, 4);
    if (mode === "year") return dateStr.slice(0, 7);
    if (mode === "month") return dateStr;
    return dateStr;
  };

  const map = new Map<string, number>();

  // Pre-fill map for 'year' (all 12 months) and 'month' (all days in month)
  if (mode === "year") {
    const y = dateFilter.value.year;
    for (let m = 1; m <= 12; m++) {
      const mm = m.toString().padStart(2, '0');
      map.set(`${y}-${mm}`, 0);
    }
  } else if (mode === "month") {
    const y = parseInt(dateFilter.value.year, 10);
    const m = parseInt(dateFilter.value.month, 10);
    const daysInMonth = new Date(y, m, 0).getDate();
    const prefix = `${dateFilter.value.year}-${dateFilter.value.month}`;
    for (let d = 1; d <= daysInMonth; d++) {
      const dd = d.toString().padStart(2, '0');
      map.set(`${prefix}-${dd}`, 0);
    }
  }

  for (const r of typeRecords) {
    const key = getKey(r.date);
    // Only add to map if it's 'all' mode, or if the key is already in our pre-filled map
    // (though typeRecords is already filtered to the selected period, so it should match)
    if (mode === "all" || map.has(key)) {
      map.set(key, (map.get(key) ?? 0) + r.amount);
    }
  }

  const entries = [...map.entries()].sort((a, b) => a[0].localeCompare(b[0]));
  const maxVal = entries.reduce((m, [, v]) => Math.max(m, v), 0);
  trendMaxValRaw.value = maxVal;
  if (maxVal === 0) return [];

  return entries.map(([key, value]) => {
    let shortLabel = key;
    if (mode === "all") {
      shortLabel = key; // year: 2024, 2025, 2026
    } else if (mode === "year") {
      // month: just number 1, 2, ..., 12
      shortLabel = String(parseInt(key.slice(5, 7), 10));
    } else if (mode === "month") {
      // day: just number 1, 2, ..., 31
      shortLabel = String(parseInt(key.slice(8, 10), 10));
    }

    return {
      key,
      shortLabel,
      value: Math.round(value),
      percent: Math.max(Math.round((value / maxVal) * 100), value > 0 ? 2 : 0),
    };
  });
});
</script>
