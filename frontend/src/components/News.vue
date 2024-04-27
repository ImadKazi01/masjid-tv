<template>
  <div class="news-container">
    <swiper :options="swiperOptions">
      <swiper-slide v-for="(item, index) in newsItems" :key="index">
        <div class="news-item">
          <img :src="item.image" :alt="item.title" />
          <h3>{{ item.title }}</h3>
          <p>{{ item.copy }}</p>
        </div>
      </swiper-slide>
    </swiper>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/swiper-bundle.css";
import newsData from "../data.json";

const newsItems = ref([]);
const swiperOptions = {
  autoplay: {
    delay: 1000,
    disableOnInteraction: false,
  },
  loop: true,
};

onMounted(() => {
  newsItems.value = newsData;

  // Uncomment the following code if you want to fetch data from an API
  // try {
  //   const response = await fetch('api/endpoint');
  //   const data = await response.json();
  //   newsItems.value = data;
  // } catch (error) {
  //   console.error('Error fetching news data:', error);
  // }
});
</script>

<style lang="scss" scoped>
.news-container {
  display: flex;
  width: 40%;
  height: 70vh;

  .news-item {
    text-align: center;

    img {
      max-width: 100%;
      height: auto;
    }

    h3 {
      margin-top: 1rem;
    }

    p {
      margin-top: 0.5rem;
    }
  }
}
</style>
