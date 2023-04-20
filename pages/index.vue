<template>
  <div
    class="relative w-full flex flex-col gap-10 min-h-screen p-20 bg-neutral-900"
  >
    <div
      class="fixed top-0 left-0 w-full text-white font-bold text-xl h-screen flex items-center justify-center z-10 bg-stone-300 bg-opacity-50"
      v-if="isPending"
    >
      ...Loading
    </div>
    <div v-for="teacher in teachersList" class="w-[600px] mx-auto">
      <NuxtLink
        :to="`/${teacher.user.id}`"
        class="w-full text-sm flex flex-col items-center justify-between overflow-hidden mx-auto cursor-pointer bg-white bg-opacity-80 hover:bg-sky-300 hover:bg-opacity-40 rounded-xl shadow hover:shadow-lg transition-all"
      >
        <div class="w-full h-60 bg-black">
          <video :src="teacher.video" class="w-full h-full" controls></video>
        </div>
        <div class="flex w-full items-center gap-10 py-5 px-3">
          <nuxt-img
            :src="teacher.user.avatar"
            class="w-14 rounded-full shadow"
          />
          <p>{{ teacher.user.name }}</p>
        </div>
      </NuxtLink>
    </div>

    <button
      v-show="pageNumber <= 5"
      @click="() => viewMore()"
      class="w-[600px] mx-auto col-span-3 text-white text-sm rounded-lg hover:text-black hover:bg-indigo-300 transition-all h-10 flex items-center justify-center bg-indigo-500"
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
    console.log(teachersList.value);
  },
  { immediate: true }
);

const viewMore = () => {
  pageNumber.value++;
  refresh();
};
</script>

<style scoped></style>
