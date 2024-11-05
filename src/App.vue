<script setup lang="ts">
import { ref, onMounted } from "vue";

async function fetchData(url: string) {
  try {
    const apiResponse = await fetch(url);
    if (!apiResponse.ok) {
      throw new Error(`HTTP error! status: ${apiResponse.status}`);
    }
    return await apiResponse.json();
  } catch (error) {
    console.error("Fetching data failed:", error);
    throw error;
  }
}

const stories = ref([]);
const error = ref(null);

onMounted(async () => {
  try {
    const ids = await fetchData(
      "https://hacker-news.firebaseio.com/v0/topstories.json"
    );
    const storyPromises = ids
      .slice(0, 10)
      .map((id) =>
        fetchData(`https://hacker-news.firebaseio.com/v0/item/${id}.json`)
      );
    stories.value = await Promise.all(storyPromises);
  } catch (err) {
    error.value = err;
  }
});
</script>

<template>
  <nav>
    <h1>
      <svg
        width="35px"
        height="35px"
        viewBox="0 0 256 256"
        version="1.1"
        xmlns="http://www.w3.org/2000/svg"
        xmlns:xlink="http://www.w3.org/1999/xlink"
        preserveAspectRatio="xMidYMid"
      >
        <g>
          <rect fill="#757cf9" x="0" y="0" width="256" height="256" />
          <path
            d="M119.373653,144.745813 L75.43296,62.4315733 L95.5144533,62.4315733 L121.36192,114.52416 C121.759575,115.452022 122.2235,116.413008 122.753707,117.407147 C123.283914,118.401285 123.747838,119.428546 124.145493,120.48896 C124.410597,120.886615 124.609422,121.251127 124.741973,121.582507 C124.874525,121.913886 125.007075,122.212123 125.139627,122.477227 C125.802386,123.802744 126.39886,125.095105 126.929067,126.354347 C127.459274,127.613589 127.923198,128.773399 128.320853,129.833813 C129.381268,127.580433 130.541078,125.1614 131.80032,122.57664 C133.059562,119.99188 134.351922,117.307747 135.67744,114.52416 L161.92256,62.4315733 L180.612267,62.4315733 L136.27392,145.739947 L136.27392,198.826667 L119.373653,198.826667 L119.373653,144.745813 Z"
            fill="#FFFFFF"
          />
        </g>
      </svg>
      Hacker News
    </h1>

    <a href="https://news.ycombinator.com/" class="button">
      Go to the real Hacker News
    </a>
  </nav>

  <div v-if="error">Error: {{ error }}</div>

  <ul v-else v-for="story in stories">
    <a :href="story.url" target="_blank">{{ story.title }}</a>
    <li :key="story.id" class="story-item">
      <div class="story-info">
        {{ story.score }} points by {{ story.by }} |
        {{ story.descendants }} comments
      </div>
    </li>
  </ul>
</template>

<style scoped></style>
