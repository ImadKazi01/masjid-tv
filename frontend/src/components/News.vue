<script setup>
import { ref, onMounted } from "vue";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Autoplay } from "swiper/modules";
import "swiper/swiper-bundle.css";

const newsItems = ref([]);
const currentBackgroundColor = ref("");

async function fetchSlideshow() {
  try {
    const response = await fetch(
      "https://script.googleusercontent.com/macros/echo?user_content_key=Z08OKKoHWj367xFbLy6PWdzMIuvVGsK9WOtE6Yrs7CZv8xnk1IOM2dFkSD79irTaaEPUfrHleG21wfwTgI_4uPs8W3T-aFjFOJmA1Yb3SEsKFZqtv3DaNYcMrmhZHmUMWojr9NvTBuBLhyHCd5hHawDD7f-ZhjdUHo0eo4bAqwscOgQ_LEHiRqOxz3jNEaSzz5QBRU34F3h_6i07jeuS36ORojFJlsM9SAns65DBbiiGHg4Tv-3CrRRXZqHQgsmygP99UgJFxyTv13CxkcP5JJjz6loM1y2-&lib=MEDXf46DmJ1Z0zvQ5xfCtIWE6ei3rOiSo"
    );
    if (!response.ok) throw new Error("Failed to fetch data");
    const data = await response.json();
    newsItems.value = data.data;
    console.log("Slideshow fetched at:", new Date().toLocaleTimeString());
    console.log("Data:", newsItems.value);
  } catch (error) {
    console.error("Error fetching news data:", error);
  }
}

const onSlideChange = (swiper) => {
  const activeIndex = swiper.realIndex;
  if (newsItems.value[activeIndex]) {
    currentBackgroundColor.value = newsItems.value[activeIndex].background;
  }
};

const swiperConfig = {
  autoplay: {
    delay: 5000,
    disableOnInteraction: false,
  },
  loop: true,
  slidesPerView: 1,
  modules: [Autoplay],
};

onMounted(fetchSlideshow);
</script>

<template>
  <div
    class="news-container"
    :style="{ backgroundColor: currentBackgroundColor }"
  >
    <Swiper v-bind="swiperConfig" @slideChange="onSlideChange">
      <SwiperSlide v-for="(item, index) in newsItems" :key="index">
        <div class="news-item">
          <h3>{{ item.title }}</h3>
          <figure class="news-image">
            <img :src="item.image" :alt="item.title" />
          </figure>
          <p v-if="item.copy">{{ item.copy }}</p>
        </div>
      </SwiperSlide>
    </Swiper>
  </div>
</template>

<style lang="scss" scoped>
.news-container {
  display: flex;
  width: 60%;
  height: auto;
  align-items: center;
}

.news-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  text-align: center;
  gap: 1rem;

  img {
    max-width: 70%;
    height: auto;
  }

  h3 {
    margin-top: 1rem;
    font-size: 3rem;
  }

  p {
    max-width: 75%;
    margin-top: 0.5rem;
    font-size: 1.4rem;
    padding: 0 3rem;
  }
}
</style>
