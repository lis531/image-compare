<script setup>
import { ref } from 'vue';

const image1 = ref(null);
const image2 = ref(null);

const handleImage1 = (e) => {
  image1.value = URL.createObjectURL(e.target.files[0]);
};

const handleImage2 = (e) => {
  image2.value = URL.createObjectURL(e.target.files[0]);
}

const handleRange = (e) => {
  const value = e.target.value;
  const img = document.querySelector('.left img');
  img.style.clipPath = `inset(0 ${100 - value}% 0 0)`;
};
</script>

<template>
  <header>
    <h1>Image comparator</h1>
  </header>
  <main>
    <div class="desc">
      <h2>Upload images</h2>
      <p>Upload two images to compare them.</p>
    </div>

    <div class="comparator">
      <input type="range" min="0" max="100" @input="handleRange" />
      <div class="left">
        <img v-if="image1" :src="image1" alt="Image 1" />
        <input v-if="!image1" type="file" accept="image/*" @change="handleImage1" />
      </div>
      <div class="right">
        <img v-if="image2" :src="image2" alt="Image 2" />
        <input v-if="!image2" type="file" accept="image/*" @change="handleImage2" />
      </div>
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
  margin: 10vh 0;
  font-size: large;
}

main {
  width: 90vw;
}

.desc {
  margin-bottom: 2rem;
  text-align: center;
}

.comparator {
  display: flex;
  position: relative;
  overflow: hidden;
  width: 100%;
  height: 100vh;
}
.comparator > input {
  position: absolute;
  z-index: 100;
  width: 100%;
}

.comparator > input::-webkit-slider-thumb {
  height: 100%;
}

.comparator > input:focus {
  cursor: ew-resize;
}

.comparator > div {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.comparator img {
  position: absolute;
  width: 100%;
  height: 100%;
}
.left img {
  z-index: 2;
  left: 0;
}
.right img {
  z-index: 1;
  right: 0;
}
</style>