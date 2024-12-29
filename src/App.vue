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
    const img = document.querySelector('.left img');
    img.style.opacity = 1;
  } else {
    const img = document.querySelector('.left img');
    const value = document.querySelector('#opacity').value;
    img.style.opacity = value / 100;
  }
};

const handleFade = (e) => {
  if(!changeFade) return;
  const value = e.target.value;
  const img = document.querySelector('.left img');
  img.style.opacity = value / 100;
};

import { onMounted } from 'vue';

onMounted(() => {
  document.querySelectorAll('.left, .right').forEach((div) => {
    div.addEventListener('click', () => {
      const input = document.createElement('input');
      input.type = 'file';
      input.accept = 'image/*';
      input.onchange = div.classList.contains('left') ? handleImage1 : handleImage2;
      input.click();
    });

    div.addEventListener('dragover', (e) => {
      e.preventDefault();
      e.stopPropagation();
      div.classList.add('drag-over');
    });

    div.addEventListener('dragleave', (e) => {
      div.classList.remove('drag-over');
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
          comparator.style.setProperty('--aspect-ratio', `${img.width / img.height}`);
          comparator.style.setProperty('--width', 'auto');
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
      <input v-if="image1 && image2" id="opacity" type="range" min="0" max="100" @input="handleFade" />
    </div>
    <div class="comparator" v-bind:style="{'width': image1 && image2 ? 'auto' : '100%', 'aspect-ratio': image1 && image2 ? 'var(--aspect-ratio)' : 'unset'}">
      <input v-if="image1 && image2" type="range" min="0" max="100" @input="handleRange" />
      <div class="comparatorMain">
        <div class="left" v-bind:style="{'border-width': image1 && image2 ? '0' : '4px'}">
          <img v-if="image1" :src="image1" v-bind:class="{'comparableFormat': image1 && image2}" alt="Image 1" />
          <div v-if="image1 && image2" class="borderImg"></div>
          <div v-if="!image1">
            <img class="icon" src="/public/icons/file.svg" alt="File" />
            <b>Drop image here</b>
            <p>or click to browse</p>
          </div>
        </div>
        <div class="right" v-bind:style="{'border-width': image1 && image2 ? '0' : '4px'}">
          <img v-if="image2" :src="image2" v-bind:class="{'comparableFormat': image1 && image2}" alt="Image 2" />
          <div v-if="!image2">
            <img class="icon" src="/public/icons/file.svg" alt="File" />
            <b>Drop image here</b>
            <p>or click to browse</p>
          </div>
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
  background-image: url("../public/icons/arrows.svg");
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
  margin: 1rem;
}

.comparatorMain > div > img {
  width: 100%;
  height: 100%;
  -webkit-user-drag: none;
  object-fit: contain;
  padding: 1rem;
}

.comparableFormat {
  position: absolute;
  padding: 0 !important;
  object-fit: unset !important;
}

.icon {
  width: 5rem !important;
  height: 5rem !important;
}

.left, .right {
  transition: border 0.3s;
  border-radius: 1rem;
  border: 4px dashed rgba(115, 115, 115, 0.5);
  cursor: pointer;
}

.drag-over {
  border: 4px dashed rgba(55, 55, 55, 0.5);
}

.left > div, .right > div {
  text-align: center;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.left > div > b, .right > div > b {
  font-size: large;
}

.left > div > p, .right > div > p {
  font-size: small;
}

.left > img.comparableFormat {
  z-index: 10;
  left: 0;
  clip-path: inset(0 50% 0 0);
}

.right > img.comparableFormat {
  right: 0;
  clip-path: inset(0 0 0 -50%);
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