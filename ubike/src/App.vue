<script setup>
import { ref, computed, watch, reactive, onMounted } from "vue";

// 修改這份 YouBike 即時資訊表，並加上
// 1. 站點名稱搜尋
// 2. 目前可用車輛 / 總停車格 的排序功能
// 3. 加入分頁功能，每頁 20 筆資料

// 欄位說明:
// https://data.taipei/dataset/detail?id=c6bc8aed-557d-41d5-bfb1-8da24f78f2fb
// sno：站點代號、 sna：場站名稱(中文)、 tot：場站總停車格、
// sbi：場站目前車輛數量、 sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

const uBikeStops = ref([]);
const filterSna = ref(""); // 搜尋場站名稱
const sortByField = ref("");
const sortOrder = ref(1);
const itemsPerPage = 10; //每10個一頁
let currentPage = ref(1); //目前頁數
const activeClass = ref("fa fa-sort-asc");
const errorClass = ref("fa fa-sort-desc");

onMounted(() => {
  fetch(
    "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json"
  )
    .then((res) => res.text())
    .then((data) => {
      uBikeStops.value = JSON.parse(data);
    });
});

const uBikeData = computed(() => {
  return uBikeStops.value.filter((stop) => stop.sna.includes(filterSna.value));
});

const sortBikeData = computed(() => {
  if (sortByField) {
    return uBikeData.value.slice().sort((a, b) => {
      return a[sortByField] - b[sortByField] * sortOrder;
      // * sortOrder
    });
  } else {
    return uBikeData.value;
  }
});

const paginatedUBikeData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return sortBikeData.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(sortBikeData.value.length / itemsPerPage);
});

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};

const goToFirstPage = () => {
  currentPage.value = 1;
};

const goToLastPage = () => {
  currentPage.value = totalPages.value;
};

//排序
const sortBy = (field) => {
  if (sortByField.value === "sbi") {
    sortOrder.value = -sortOrder.value;
    console.log("sbi", sortOrder.value);
  }
  if (sortByField.value === "tot") {
    sortOrder.value = -sortOrder.value;
    console.log("tot", sortOrder.value);
  } else {
    sortByField.value = field;
    sortOrder.value = 1;
  }
};

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, "$1/$2/$3 $4:$5:$6");
};
</script>


<template>
  <div class="app">
    <p>
      站點名稱搜尋: <input v-model="filterSna" placeholder="輸入場站名稱" />
    </p>
    <table class="table table-striped">
      <thead>
        <tr>
          <th>#</th>
          <th @click="sortBy('sna')">場站名稱</th>
          <th>場站區域</th>
          <th @click="sortBy('sbi')" style="cursor: pointer">
            目前可用車輛
            <i :class="[sortByField === 'sbi' ? activeClass : errorClass]"> </i>
          </th>
          <th @click="sortBy('tot')" style="cursor: pointer">
            總停車格
            <i :class="[sortByField === 'tot' ? activeClass : errorClass]"> </i>
          </th>
          <th>資料更新時間</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(s, index) in paginatedUBikeData" :key="s.sno">
          <td>{{ index + 1 }}</td>
          <td>{{ s.sna }}</td>
          <td>{{ s.sarea }}</td>
          <td>{{ s.sbi }}</td>
          <td>{{ s.tot }}</td>
          <td>{{ timeFormat(s.mday) }}</td>
        </tr>
      </tbody>
    </table>
    <button @click="goToFirstPage" :disabled="currentPage === 1">First</button>
    <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
    <span v-for="page in totalPages" :key="page.id">{{ page }}</span>
    <button>{{ currentPage }}</button>

    <button @click="nextPage" :disabled="currentPage === totalPages">
      Next
    </button>
    <button @click="goToLastPage" :disabled="currentPage === totalPages">
      Last
    </button>
    Total:{{ totalPages }}
  </div>
</template>


<style scoped>
.app {
  padding: 1rem;
}
</style>