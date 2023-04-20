<template>
  <div class="min-h-screen p-20 bg-slate-200">
    <div
      v-if="!selectedPlan"
      class="flex items-center justify-center gap-5 p-20"
    >
      <div
        @click="() => (selectedPlan = 1)"
        class="w-72 aspect-square text-xl drop-shadow rounded shadow hover:text-white transition-all cursor-pointer hover:scale-105 hover:bg-rose-400 bg-white flex items-center justify-center"
      >
        نیم ساعته
      </div>
      <div
        @click="() => (selectedPlan = 2)"
        class="w-72 aspect-square text-xl drop-shadow rounded shadow hover:text-white transition-all cursor-pointer hover:scale-105 hover:bg-blue-400 bg-white flex items-center justify-center"
      >
        یک ساعته
      </div>
    </div>

    <div v-else>
      <div class="relative flex justify-center gap-16">
        <button
          @click="() => selectedWeek--"
          :disabled="!selectedWeek"
          class="bg-blue-950 text-white text-xs px-10 py-2 rounded absolute -top-10 right-0 disabled:opacity-30"
        >
          هفته قبل
        </button>
        <button
          @click="() => selectedWeek++"
          class="bg-blue-950 text-white text-xs px-10 py-2 rounded absolute -top-10 left-0"
        >
          هفته بعد
        </button>
        <div
          v-for="(day, dayIndex) in weekList"
          :key="day"
          class="w-40 group min-h-fit flex flex-col bg-white outline shadow outline-zinc-300 rounded-lg py-5 items-center"
        >
          <p class="border-b w-full text-center pb-3 mb-3">
            {{ dayDefiner(day[0].timestamp) }}
          </p>
          <button
            v-for="(time, timeIndex) in day"
            class="relative w-full flex flex-col h-9 mx-3 py-2 disabled:opacity-30 disabled:cursor-not-allowed transition-all rounded-xl"
            :class="checkAvailable(dayIndex, timeIndex)"
            @click="() => selectHandler(time.id)"
            :key="time.timestamp"
            :disabled="time.status === 'reserved'"
          >
            <div
              class="w-full text-center h-full flex items-center justify-center flex-row-reverse"
            >
              <span>{{ new Date(time.timestamp * 1000).getHours() }}</span>
              <span>:</span>
              <span>{{
                new Date(time.timestamp * 1000).getMinutes() || "00"
              }}</span>
            </div>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const { id } = useRoute().params;
const selectedPlan = ref(0);
const selectedWeek = ref(0);
const startTime = Math.trunc(new Date().getTime() / 1000); //+604800000 ms
const weekList: any = ref([]);
let allTimes;
const now = new Date().getDate();
const selectedId = ref(0);

// Fetching Data ------------------------------------
const { data, refresh } = useFetch(
  `https://api.hitalki.org/api/users/${id}/schedule?from=${startTime}&to=1682452800`
);

// Modifing Fetched Data ------------------------------------
watch(data, () => {
  allTimes = JSON.parse(JSON.stringify(data))._value;

  let startSclice: number = now;
  for (let i = 0; i < allTimes.data.length; i++) {
    const dayOfClass = new Date(allTimes.data[i].timestamp * 1000).getDate();

    if (dayOfClass - startSclice > 0) {
      const week = allTimes.data.splice(0, i);
      weekList.value.push(week);
      startSclice = dayOfClass;
      i = 0;
    } else if (i === allTimes.data.length - 1) {
      const lastWeek = allTimes.data.splice(0, i);
      weekList.value.push(lastWeek);
    }
  }
});

// Hours Select Classes --------------------------------
const checkAvailable = (dayIndex: number, timeIndex: number) => {
  if (
    selectedPlan.value === 2 &&
    weekList.value[dayIndex][timeIndex]?.status === "available" &&
    weekList.value[dayIndex][timeIndex + 1]?.status === "available"
  ) {
    if (selectedId.value === weekList.value[dayIndex][timeIndex]?.id) {
      return "two-select clicked";
    }
    return "two-select";
  }

  if (selectedPlan.value === 1) {
    if (selectedId.value === weekList.value[dayIndex][timeIndex]?.id) {
      return "single-selected single-clicked";
    }
    return "single-selected";
  }
};

// Define the Day of the Week ---------------------------
const dayDefiner = (timeStamp: number) => {
  const date = new Date(timeStamp * 1000).getDay();
  switch (date) {
    case 0:
      return "یکشنبه";

    case 1:
      return "دوشنبه";

    case 2:
      return "سه شنبه";

    case 3:
      return "چهارشنبه";

    case 4:
      return "پنجشنبه";

    case 5:
      return "جمعه";

    case 6:
      return "شنبه";
  }
};

// Select an hour -----------------------------------------
const selectHandler = (id: number) => {
  if ((selectedId.value = id)) {
    selectedId.value = 0;
  }
  selectedId.value = id;
};
</script>

<style scoped>
.two-select:hover::after {
  content: "";
  position: absolute;
  top: 10%;
  left: 0;
  width: calc(100% - 24px);
  height: 180%;
  margin: 0 12px;
  opacity: 50%;
  border: 2px solid #4f4cec;
  border-radius: 10px;
  z-index: 10;
}

.clicked::after {
  content: "";
  position: absolute;
  top: 10%;
  left: 0;
  width: calc(100% - 24px);
  height: 180%;
  margin: 0 12px;
  opacity: 50%;
  background: #4f4cec;
  border-radius: 10px;
  z-index: 10;
}
.single-clicked::after {
  content: "";
  position: absolute;
  top: 0%;
  left: 0;
  width: calc(100% - 24px);
  height: 100%;
  margin: 0 12px;
  opacity: 50%;
  background: #4f4cec;
  border-radius: 10px;
  z-index: 10;
}

.single-selected:hover::after {
  content: "";
  position: absolute;
  top: 0%;
  left: 0;
  width: calc(100% - 24px);
  height: 100%;
  margin: 0 12px;
  opacity: 50%;
  border: 2px solid #4f4cec;
  border-radius: 10px;
  z-index: 10;
}
</style>
