<script setup>
import { ref } from 'vue';

const image1 = ref('');
const image2 = ref('');

const handleImage1 = (e) => {
  image1.value = URL.createObjectURL(e.target.files[0]);
  resizeComparator();
};

const handleImage2 = (e) => {
  image2.value = URL.createObjectURL(e.target.files[0]);
  resizeComparator();
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
  if (!changeFade) return;
  const value = e.target.value;
  const img = document.querySelector('.left img');
  img.style.opacity = value / 100;
};

const resizeComparator = () => {
  const img1 = new Image();
  const img2 = new Image();
  img1.src = image1.value;
  img2.src = image2.value;

  const comparator = document.querySelector('.comparator');

  const onLoad = () => {
    if (!img1.complete || !img2.complete) return;
    const img = img1.width * img1.height > img2.width * img2.height ? img1 : img2;
    comparator.style.setProperty('--aspect-ratio', `${img.width} / ${img.height}`);
    comparator.style.setProperty('--height', `${img.height}px`);
  };

  img1.onload = onLoad;
  img2.onload = onLoad;
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
      const input = document.querySelector('.comparator > input');
      input.style.visibility = 'hidden';
    });

    div.addEventListener('dragleave', (e) => {
      div.classList.remove('drag-over');
      const input = document.querySelector('.comparator > input');
      input.style.visibility = 'visible';
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
      resizeComparator();
      const input = document.querySelector('.comparator > input');
      input.style.visibility = 'visible';
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
    <div v-if="image1 && image2" class="opacityControls">
      <div>
        <label>Opacity?</label>
        <input type="checkbox" @change="handleSwitch" />
      </div>
      <input id="opacity" type="range" min="0" max="100" @input="handleFade" />
    </div>
    <div class="comparator" v-bind:style="{'width': image1 && image2 ? 'auto' : '100%', 'height': image1 && image2 ? 'var(--height)' : '75vh', 'aspect-ratio': image1 && image2 ? 'var(--aspect-ratio)' : 'unset'}">
      <input v-if="image1 && image2" type="range" min="0" max="100" @input="handleRange" />
      <div class="comparatorMain" v-bind:style="{'flex-direction': image1 && image2 ? 'row !important' : 'initial'}">
        <div class="left" v-bind:style="{'border-width': image1 && image2 ? '0' : '4px'}">
          <img v-if="image1" :src="image1" v-bind:class="{'comparableFormat': image1 && image2}" alt="Image 1" />
          <div v-else>
            <img class="icon" src="/src/icons/file.svg" alt="File" />
            <b>Drop image here</b>
            <p>or click to browse</p>
          </div>
          <div v-if="image1 && image2" class="borderImg"></div>
        </div>
        <div class="right" v-bind:style="{'border-width': image1 && image2 ? '0' : '4px'}">
          <img v-if="image2" :src="image2" v-bind:class="{'comparableFormat': image1 && image2}" alt="Image 2" />
          <div v-else>
            <img class="icon" src="/src/icons/file.svg" alt="File" />
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
  margin: 3vh 0 1vh;
  font-size: large;
}

main {
  width: 90vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

p, label {
  font-size: medium;
}

.desc {
  margin-bottom: 4vh;
  text-align: center;
}

.opacityControls {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  margin-bottom: 2rem;
}

.opacityControls > input[type="range"] {
  width: 400px;
  height: 30px;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background: none;
  cursor: pointer;
  overflow: hidden;
}

.opacityControls > input[type="range"]::-webkit-slider-thumb {
  height: 30px;
  width: 30px;
  background: none;
  aspect-ratio: 1;
  border-radius: 50%;
  box-shadow: 0 0 0 var(--fill, 6px) #40c9ff inset;
  border-image: linear-gradient(90deg, #40c9ff 50%, #ababab 0) 0 1/calc(50% - 6px/2) 100vw/0 calc(100vw + 8px);
  -webkit-appearance: none;
  appearance: none;
  transition: box-shadow 0.3s;
}

.opacityControls > input[type="range"]::-moz-range-thumb {
  height: 30px;
  width: 30px;
  background: none;
  aspect-ratio: 1;
  border-radius: 50%;
  box-shadow: 0 0 0 var(--fill, 6px) #40c9ff inset;
  border-image: linear-gradient(90deg, #40c9ff 50%, #ababab 0) 0 1/calc(50% - 6px/2) 100vw/0 calc(100vw + 8px);
  -moz-appearance: none;
  appearance: none;
  transition: box-shadow 0.3s;
}

.opacityControls > input[type="range"]:active, .opacityControls > input[type="range"]:focus-visible {
  --fill: 30px;
}

.opacityControls > div {
  display: flex;
  align-items: center;
  margin-bottom: 1rem;
}

.opacityControls > div > label {
  margin-right: 1rem;
}

.opacityControls > div > input[type="checkbox"] {
  width: 15px;
  height: 15px;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border: 2px solid #40c9ff;
}

.opacityControls > div > input[type="checkbox"]:checked {
  background-image: url("/src/icons/check.svg");
  background-color: #40c9ff;
  background-position: center;
}

.comparator {
  display: flex;
  align-items: center;
  position: relative;
  max-height: 72vh;
  max-width: 100%;
  border-radius: 0.3rem;
  overflow: visible;
}

.comparator > input {
  position: absolute;
  z-index: 100;
  width: calc(100% + 35px);
  appearance: none;
  -webkit-appearance: none;
  background-color: transparent;
  left: -17.5px;
}

.comparator > input::-webkit-slider-thumb {
  cursor: ew-resize;
  -webkit-appearance: none;
  appearance: none;
  width: 35px;
  height: 35px;
  border-radius: 50%;
  background-image: url("icons/arrows.svg");
  background-color: rgba(28, 28, 28, 0.5);
  background-size: 100%;
  border: 2px solid rgba(20, 20, 20, 0.5);
}

.comparator > input::-moz-range-thumb {
  cursor: ew-resize;
  -moz-appearance: none;
  appearance: none;
  width: 35px;
  height: 35px;
  border-radius: 50%;
  background-image: url("icons/arrows.svg");
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
  @media (max-width: 768px) {
    flex-direction: column !important;
  }
}

.comparatorMain > div {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 50%;
  height: 100%;
  margin: 1rem;
  @media (max-width: 768px) {
    width: 100%;
  }
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
</style>