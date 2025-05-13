<template>
  <div class="text-white p-4">
    <div v-if="error" class="text-red-400 p-4 bg-red-900/20 rounded-lg">
      {{ error }}
    </div>
    <div v-else-if="!data" class="text-gray-400 p-4">Загрузка данных...</div>

    <div v-if="data" class="space-y-6">
      <div class="flex flex-wrap gap-3">
        <button
          v-for="(action, index) in actions"
          :key="index"
          class="cursor-pointer flex w-11 h-11 items-center justify-center bg-[#111113] hover:bg-[#080809] active:bg-gray-600 rounded-xl transition"
          :aria-label="action.label"
          :title="action.label"
        >
          <component :is="action.icon" class="w-5 h-5 text-white" />
        </button>
      </div>

      <div class="flex flex-col md:flex-row gap-6">
        <div
          class="w-full md:w-64 h-64 bg-black flex items-center justify-center rounded-2xl overflow-hidden shrink-0"
        >
          <img
            :src="'https://dev.jobcart.ru' + data.photo"
            alt="Фото кандидата"
            class="w-full h-full object-cover"
            loading="lazy"
          />
        </div>

        <div class="flex-1 space-y-3">
          <h1 class="text-2xl font-bold">{{ data.name }}</h1>

          <div class="space-y-2">
            <p>
              Кандидат на вакансию:
              <a
                :href="`https://dev.jobcart.ru/vacancy/${data.listing_id}`"
                target="_blank"
                class="text-blue-400 hover:text-blue-300 inline-flex items-center gap-1"
              >
                {{ data.vacancy_name || "Frontend разработчик" }}
                <i-heroicons-phone class="w-4 h-4" />
              </a>
              <span class="text-gray-400 text-sm"
                >({{ data.date }} Отклик)</span
              >
            </p>

            <p class="text-gray-400">Просмотрено</p>

            <div class="flex flex-wrap gap-x-4 gap-y-2">
              <p>
                Возраст: <span class="text-white">{{ data.age }}</span>
              </p>
              <p>
                Город: <span class="text-white">{{ data.town }}</span>
              </p>
            </div>

            <div class="flex flex-wrap gap-4 pt-1">
              <a
                :href="`tel:${data.phone}`"
                class="flex items-center gap-1 hover:text-blue-300"
              >
                <Telephone class="w-4 h-4" />
                {{ formatPhone(data.phone) }}
              </a>
              <a
                :href="`mailto:${data.email}`"
                class="flex items-center gap-1 hover:text-blue-300"
              >
                <AtSymbolIcon class="w-4 h-4" />
                {{ data.email }}
              </a>
            </div>
          </div>
        </div>
      </div>

      <div class="space-y-4">
        <h2 class="text-lg font-semibold">Дела:</h2>
        <div class="flex flex-wrap gap-3">
          <button
            v-for="(task, index) in tasks"
            :key="index"
            class="cursor-pointer px-4 py-2 w-full sm:w-[175px] sm:h-[68px] text-white border-2 border-green-500 rounded-md transition-all hover:bg-green-500 focus:outline-none focus:ring-2 focus:ring-green-400 disabled:opacity-50 disabled:cursor-not-allowed"
            :disabled="task.disabled"
            :style="{ borderColor: '#00c950' }"
          >
            {{ task.label }}
          </button>
        </div>
      </div>

      <div class="space-y-4">
        <h2 class="text-lg font-semibold">Статус рассмотрения</h2>
        <div class="flex flex-wrap gap-2">
          <div
            v-for="(status, index) in computedStatuses"
            :key="index"
            class="px-3 py-3 rounded-lg text-sm font-medium transition-all"
            :class="{
              'bg-green-500 text-white shadow-md': status.current,
              'bg-[#111113] text-gray-400 hover:bg-[#080809]': !status.current,
            }"
          >
            {{ status.label }}
          </div>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <h4 class="text-lg font-semibold">Pikadu отклик</h4>
          <p class="text-gray-300">Отклик с портала pikadu.</p>
        </div>

        <div class="space-y-2">
          <p class="text-gray-300">
            Дата рождения:
            <span class="text-white font-semibold">{{
              formatDate(data.birth_date)
            }}</span>
          </p>
          <p class="text-gray-300">
            Гражданство:
            <span class="text-white font-semibold">Россия</span>
          </p>
        </div>
      </div>

      <div class="space-y-2">
        <h2 class="text-xl font-bold">Сопроводительное письмо:</h2>
        <div class="bg-[#111113] p-4 rounded-lg">
          <pre class="text-sm whitespace-pre-line font-sans">{{
            data.description
          }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
import AtSymbolIcon from "@heroicons/vue/24/outline/AtSymbolIcon";
import Telephone from "@heroicons/vue/24/outline/DevicePhoneMobileIcon";
import DocumentCurrencyRupeeIcon from "@heroicons/vue/24/outline/DocumentCurrencyRupeeIcon";
import DocumentMagnifyingGlassIcon from "@heroicons/vue/24/outline/DocumentMagnifyingGlassIcon";
import Document from "@heroicons/vue/24/outline/DocumentIcon";
import UserGroupIcon from "@heroicons/vue/24/outline/UserGroupIcon";
import ArchiveBoxXMarkIcon from "@heroicons/vue/24/outline/ArchiveBoxXMarkIcon";
import Submit from "@heroicons/vue/24/outline/DocumentArrowUpIcon";
import StarIcon from "@heroicons/vue/24/outline/StarIcon";

const formatPhone = (phone: string) => {
  if (!phone || phone.length < 11) return phone;
  const digits = phone.replace(/\D/g, "");
  return `+7 (${digits.slice(1, 4)}) ${digits.slice(4, 7)}-${digits.slice(
    7,
    9
  )}-${digits.slice(9, 11)}`;
};

const formatDate = (dateString: string): string => {
  const [year, month, day] = dateString.split("-");
  return `${day}.${month}.${year}`;
};

const id = 3832575;
const apiUrl = `https://dev.jobcart.ru/wp-json/test/v2/app?id=${id}`;

// Запрашиваем данные
const { data, error } = await useFetch(apiUrl);

// Базовые статусы
const baseStatuses = [
  { label: "Новое", key: "new" },
  { label: "Просмотрено", key: "viewed" },
  { label: "Ожидание ответа", key: "pending" },
  { label: "Принятие решения", key: "decision_pending" },
  { label: "Отправлено приглашение", key: "invitation_sent" },
  { label: "Принят", key: "accepted" },
  { label: "Назначено собеседование", key: "interview_scheduled" },
  { label: "Не дошел", key: "did_not_attend" },
  { label: "Проведено собеседование", key: "interview_done" },
  { label: "Отклонено/Отказ", key: "rejected" },
  { label: "Архивировано", key: "archived" },
];

const computedStatuses = computed(() => {
  const currentStatus = data.value?.status;
  return baseStatuses.map((status) => ({
    ...status,
    current:
      currentStatus === "viewed" && ["new", "viewed"].includes(status.key),
  }));
});

const actions = [
  { icon: Document, label: "Пригласить" },
  { icon: DocumentCurrencyRupeeIcon, label: "Собеседование" },
  { icon: DocumentMagnifyingGlassIcon, label: "Принять" },
  { icon: UserGroupIcon, label: "Отклонить" },
  { icon: ArchiveBoxXMarkIcon, label: "В архив" },
  { icon: Submit, label: "Напомнить" },
  { icon: StarIcon, label: "Контакт" },
];

const tasks = [
  { label: "Собеседование запланировано", disabled: false },
  { label: "Создать видеозвонок", disabled: false },
  { label: "Запланировать событие", disabled: true },
  { label: "Отправить запрос", disabled: false },
];
</script>
