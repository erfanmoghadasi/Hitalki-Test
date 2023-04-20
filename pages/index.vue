<template>
  <div
    class="relative w-full grid grid-cols-3 gap-10 min-h-screen p-20 bg-slate-200"
  >
    <div
      class="fixed top-0 left-0 w-full h-screen flex items-center justify-center z-10 bg-stone-300 bg-opacity-50"
      v-if="isPending"
    >
      pending
    </div>
    <div v-for="teacher in teachersList" class="">
      <NuxtLink
        :to="`/${teacher.user.id}`"
        class="w-full text-sm flex items-center justify-between p-3 overflow-hidden mx-auto cursor-pointer bg-white hover:bg-red-300 rounded-xl shadow hover:shadow-lg transition-all"
      >
        <nuxt-img :src="teacher.user.avatar" class="w-14 rounded-full shadow" />
        <p>{{ teacher.user.name }}</p>
      </NuxtLink>
    </div>

    <button
      v-show="pageNumber <= 6"
      @click="() => viewMore()"
      class="w-full col-span-3 text-white text-sm rounded-lg hover:text-black hover:bg-indigo-300 transition-all h-10 flex items-center justify-center bg-indigo-500"
    >
      اساتید بیشتر
    </button>
  </div>
</template>

<script setup lang="ts">
const teachersList: any = ref([]);
const pageNumber = ref(1);
const isPending = ref(true);
let newData;

const pageFetch = computed(() => {
  return `https://api.hitalki.org/api/pros?filters%5Btop_category_id%5D=1&top_category_id=1&page=${pageNumber.value}`;
});

const { data, pending, refresh, error } = await useFetch(pageFetch);

watch(
  pending,
  () => {
    isPending.value = pending.value;
  },
  { immediate: true }
);

watch(
  data,
  () => {
    newData = JSON.parse(JSON.stringify(data))._value;
    teachersList.value = teachersList.value.concat(newData.data);
  },
  { immediate: true }
);

const viewMore = () => {
  pageNumber.value++;
  refresh();
};
</script>

<style scoped></style>
