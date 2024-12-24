<script setup>
import { ref } from 'vue';

const image1 = ref('');
const image2 = ref('');

const handleImage1 = (e) => {
  image1.value = URL.createObjectURL(e.target.files[0]);
  const img = new Image();
  img.src = image1.value;
  img.onload = () => {
    const comparator = document.querySelector('.comparator');
    comparator.style.aspectRatio = `${img.width / img.height}`;
    comparator.style.width = 'auto';
  };
};

const handleImage2 = (e) => {
  image2.value = URL.createObjectURL(e.target.files[0]);
}

const handleRange = (e) => {
  const value = e.target.value;
  const img1 = document.querySelector('.left img');
  const img2 = document.querySelector('.right img');
  const borderImg = document.querySelector('.borderImg');
  img1.style.clipPath = `inset(0 ${100 - value}% 0 0)`;
  img2.style.clipPath = `inset(0 0 0 ${value - 100}%)`;
  borderImg.style.width = `calc(${value}% + 2px)`;
};

let changeFade = false;

const handleSwitch = (e) => {
  changeFade = e.target.checked;
  if(!changeFade) {
    const img1 = document.querySelector('.left img');
    const img2 = document.querySelector('.right img');
    img1.style.opacity = 1;
    img2.style.opacity = 1;
  } else {
    const img1 = document.querySelector('.left img');
    const img2 = document.querySelector('.right img');
    const value = document.querySelector('#opacity').value;
    img1.style.opacity = value / 100;
    img2.style.opacity = 1 - value / 100;
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
        const img = new Image();
        img.src = image1.value;
        img.onload = () => {
          const comparator = document.querySelector('.comparator');
          comparator.style.aspectRatio = `${img.width / img.height}`;
          comparator.style.width = 'auto';
        };
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
      <p>Upload two images to compare them.</p>
    </div>
    <div class="opacityControls">
      <label v-if="image1 && image2">Opacity?</label>
      <input v-if="image1 && image2" type="checkbox" @change="handleSwitch" />
      <input id="opacity" v-if="image1 && image2" type="range" min="0" max="100" @input="handleFade" />
    </div>
    <div class="comparator">
      <input v-if="image1 && image2" type="range" min="0" max="100" @input="handleRange" />
      <div class="comparatorMain">
        <div class="left">
          <img v-if="image1" :src="image1" alt="Image 1" />
          <div v-if="image1" class="borderImg"></div>
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
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.desc {
  margin-bottom: 2rem;
  text-align: center;
}

.opacityControls {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 2rem;
}

.comparator {
  display: flex;
  align-items: center;
  position: relative;
  height: 75vh;
  width: 100%;
  max-width: 100%;
  border-radius: 0.3rem;
  overflow: visible;
}

.comparator > input {
  position: absolute;
  z-index: 100;
  width: calc(100% + 35px);
  -webkit-appearance: none;
  background: transparent;
  left: -17.5px;
}

.comparator > input::-webkit-slider-thumb {
  cursor: ew-resize;
  -webkit-appearance: none;
  appearance: none;
  width: 35px;
  height: 35px;
  border-radius: 50%;
  background-image: url("public/arrows.svg");
  background-color: rgba(28, 28, 28, 0.5);
  background-size: 100%;
  border: 2px solid rgba(20, 20, 20, 0.5);
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

.borderImg {
  position: absolute;
  width: 50%;
  height: 100%;
  border-right: 2px solid rgba(28, 28, 28, 0.5);
  left: 0;
  z-index: 15;
}

.left > input, .right > input {
  z-index: 1000;
}
</style>