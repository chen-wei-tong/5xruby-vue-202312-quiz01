<script setup>
/** 
完成以下功能：
1. 將 movies 的內容用 v-for 改寫成動態渲染
2. 完成電影星星評分的事件綁定，同樣用 v-for 進行改寫
*/
import { ref, reactive, computed, onMounted } from "vue";
import { StarIcon } from "@heroicons/vue/24/solid";
import { items } from "./movies.json";

const movies = ref(items);
// const genres = reactive({});
// const rating = ref(0);

const rating = computed(() => (ratingHover.value = stars.rating));

const ratingHover = ref();

const ratingToggle = (value, id) => {
  stars.rating = value;
  console.log(`評等：${stars.rating}`);
};
const stars = reactive({
  rating: "",
});
//預設rating4,4,3
const setHoverRating = (value, id) => {
  items.forEach((item, index) => {
    ratingHover.value = value;
    // value = items.item.rating
    console.log("ratingHover", value);
  });
};
const resetHoverRating = () => {
  ratingHover.value = null;
};

onMounted(() => {
  if (items && items.length > 0) {
    // for (let i = 0; i < items.length; i++) {
    //   console.log(`Movie ${i + 1} Rating: ${items[i].rating}`);
    //   console.log("i", items[i].rating);
    // }
    items.forEach((item, index) => {
      console.log("預設?顆星", item.rating, index);
    });
  }
});
</script>

<template>
  <div class="app">
    <div class="movie-list">
      <div class="movie-item" v-for="movie in movies" :key="movie.id">
        <div class="movie-item-image-wrapper">
          <img :src="movie.image" class="movie-item-image" alt="movie" />
        </div>
        <div class="movie-item-content-wrapper">
          <div class="movie-item-title-wrapper">
            <h3 class="movie-item-title">
              {{ movie.name }}
            </h3>
            <div class="movie-item-genres-wrapper">
              <span
                v-for="(genre, index) in movie.genres"
                :key="index"
                class="movie-item-genre-tag"
              >
                {{ genre }}
              </span>
            </div>
          </div>
          <div class="movie-item-description-wrapper">
            <p class="movie-item-description">
              {{ movie.description }}
            </p>
          </div>
          <div class="movie-item-rating-wrapper">
            {{ movie.rating }}
            <span class="movie-item-rating-text">
              Rating: ({{ stars.rating }}/5)
            </span>

            <div class="movie-item-star-icon-wrapper">
              <button
                v-for="(i, index) in 5"
                :key="i.id"
                class="movie-item-star-icon-button"
                :class="[
                  i <= (ratingHover || stars.rating)
                    ? 'text-yellow-500'
                    : 'text-gray-500',
                ]"
                @mouseover="setHoverRating(i, movie.id)"
                @mouseout="resetHoverRating"
                @click="ratingToggle(i, movie.id)"
              >
                <StarIcon class="movie-item-star-icon" />
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
