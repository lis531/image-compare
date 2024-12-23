<script setup>
import { ref } from 'vue';

const image1 = ref('');
const image2 = ref('');

const handleImage1 = (e) => {
  image1.value = URL.createObjectURL(e.target.files[0]);
};

const handleImage2 = (e) => {
  image2.value = URL.createObjectURL(e.target.files[0]);
}

const handleRange = (e) => {
  const value = e.target.value;
  const img1 = document.querySelector('.left img');
  const img2 = document.querySelector('.right img');
  img1.style.clipPath = `inset(0 ${100 - value}% 0 0)`;
  img2.style.clipPath = `inset(0 0 0 ${value - 100}%)`;
};

let changeFade = false;

const handleSwitch = (e) => {
  changeFade = e.target.checked;
  if(!changeFade) {
    const img1 = document.querySelector('.left img');
    const img2 = document.querySelector('.right img');
    img1.style.opacity = 1;
    img2.style.opacity = 1;
  }
};

const handleFade = (e) => {
  if(!changeFade) return;
  const value = e.target.value;
  const img1 = document.querySelector('.left img');
  const img2 = document.querySelector('.right img');
  img1.style.opacity = value / 100;
  img2.style.opacity = 1 - value / 100;
};

import { onMounted } from 'vue';

onMounted(() => {
  document.querySelectorAll('.left, .right').forEach((div) => {
    div.addEventListener('dragover', (e) => {
      e.preventDefault();
      e.stopPropagation();
    });

    div.addEventListener('drop', (e) => {
      e.preventDefault();
      e.stopPropagation();
      const file = e.dataTransfer.files[0];
      if (div.classList.contains('left')) {
        image1.value = URL.createObjectURL(file);
      } else {
        image2.value = URL.createObjectURL(file);
      }
    });
  });
});

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
    <label>Opacity?</label>
    <input type="checkbox" @change="handleSwitch" />
    <input type="range" min="0" max="100" @input="handleFade" />
    <div class="comparator">
      <input type="range" min="0" max="100" @input="handleRange" />
      <div class="comparatorMain">
          <div class="left">
            <img v-if="image1" :src="image1" alt="Image 1" />
            <input v-if="!image1" type="file" accept="image/*" @change="handleImage1" />
          </div>
        <div class="right">
          <img v-if="image2" :src="image2" alt="Image 2" />
          <input v-if="!image2" type="file" accept="image/*" @change="handleImage2" />
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
  margin: 3vh 0;
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
  height: 75vh;
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

.comparatorMain {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
}

.comparatorMain > div {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 50%;
  height: 100%;
}

.comparator img {
  position: absolute;
  width: 100%;
  height: 100%;
  -webkit-user-drag: none;
}

.left img {
  z-index: 10;
  left: 0;
  clip-path: inset(0 50% 0 0);
}

.right img {
  right: 0;
  clip-path: inset(0 0 0 50%);
}


.left > input, .right > input {
  z-index: 1000;
}
</style>