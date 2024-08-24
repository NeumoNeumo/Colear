<script setup lang="ts">
import { onMounted, ref, Ref } from "vue";

interface Color {
  r: number;
  g: number;
  b: number;
  a: number;
}

type State = "add" | "remove" | "normal";

const magnifierData: Ref<Color[][]> = ref(
  new Array(11)
    .fill(null)
    .map(() => new Array(11).fill({ r: 0, g: 0, b: 0, a: 0 }))
);
const imageCanvas = ref<HTMLCanvasElement | null>(null);
const ctx = ref<CanvasRenderingContext2D | null>(null);
const magnifier = ref<HTMLDivElement | null>(null);
const selectedColor = ref<Color>({ r: 0, g: 0, b: 0, a: 0 });
const paletteData = ref<Color[]>([]);
const selectedN = ref<number>(-1);
let imageSrcArray: Uint8ClampedArray | null = null;
let state = ref<State>("normal");
// const K = 5;

function getGrayScale(color: Color): number {
  return 0.299 * color.r + 0.587 * color.g + 0.114 * color.b;
}

function isBright(color: Color): boolean {
  return (getGrayScale(color) * color.a) / 255.0 > 127;
}

function updataImageSrcData() {
  if (!ctx.value) return;
  imageSrcArray = ctx.value.getImageData(
    0,
    0,
    ctx.value.canvas.width,
    ctx.value.canvas.height
  ).data;
}

function handleAdd() {
  selectedN.value = -1;
  refreshCanvas();
  if (state.value == "add") state.value = "normal";
  else state.value = "add";
}

function handleRemove() {
  selectedN.value = -1;
  refreshCanvas();
  if (state.value == "remove") state.value = "normal";
  else state.value = "remove";
}

function refreshPaletteData() {
  updataImageSrcData();
  if (!imageSrcArray) return;
  // let centroids: Color[] = [];

  // for (let i = 0; i < K; i++) {
  //   const randomInt = Math.floor((Math.random() * imageSrcArray.length) / 4);
  //   centroids.push({
  //     r: imageSrcArray[randomInt * 4],
  //     g: imageSrcArray[randomInt * 4 + 1],
  //     b: imageSrcArray[randomInt * 4 + 2],
  //     a: 255,
  //   });
  // }

  // let clusters: Color[][] = Array.from({ length: K }, () => []);
  // let population: number[] = Array.from({ length: K }, () => 0);
  // while (true) {
  //   for (let i = 0; i < imageSrcArray.length / 4; i++) {
  //     let closestCentroidIndex = -1;
  //     let minDist2 = Infinity;
  //     for (let j = 0; j < K; j++) {
  //       const dist2 =
  //         Math.pow(imageSrcArray[4 * i] - centroids[j].r, 2) +
  //         Math.pow(imageSrcArray[4 * i + 1] - centroids[j].b, 2) +
  //         Math.pow(imageSrcArray[4 * i + 2] - centroids[j].g, 2);
  //       if (dist2 < minDist2) {
  //         minDist2 = dist2;
  //         closestCentroidIndex = j;
  //       }
  //     }
  //     clusters[closestCentroidIndex].push({
  //       r: imageSrcArray[i * 4],
  //       g: imageSrcArray[i * 4 + 1],
  //       b: imageSrcArray[i * 4 + 2],
  //       a: 255,
  //     });
  //   }

  //   let newCentroids: Color[] = [];
  //   for (let i = 0; i < K; i++) {
  //     population[i] = clusters[i].length;
  //   }
  //   for (let i = 0; i < K; i++) {
  //     if (clusters[i].length > 0) {
  //       const sumR = clusters[i].reduce((sum, color) => sum + color.r, 0);
  //       const sumG = clusters[i].reduce((sum, color) => sum + color.g, 0);
  //       const sumB = clusters[i].reduce((sum, color) => sum + color.b, 0);

  //       newCentroids.push({
  //         r: sumR / clusters[i].length,
  //         g: sumG / clusters[i].length,
  //         b: sumB / clusters[i].length,
  //         a: 255,
  //       });
  //     } else {
  //       newCentroids.push(centroids[i]);
  //     }
  //   }
  //   if (isConverged(centroids, newCentroids)) break;
  //   centroids = newCentroids;
  // }
  // paletteData.value = centroids
  //   .map((value, index) => ({ value, order: population[index] }))
  //   .sort((a, b) => b.order - a.order)
  //   .map(({ value }) => value);
}

// function isConverged(centroids: Color[], newCentroids: Color[]) {
//   const epsilon = 1;
//   for (let i = 0; i < centroids.length; i++) {
//     if (
//       Math.abs(centroids[i].r - newCentroids[i].r) > epsilon &&
//       Math.abs(centroids[i].g - newCentroids[i].g) > epsilon &&
//       Math.abs(centroids[i].b - newCentroids[i].b) > epsilon
//     )
//       return false;
//   }
//   return true;
// }

onMounted(() => {
  const _ctx = imageCanvas.value?.getContext("2d", {
    willReadFrequently: true,
  });
  if (!_ctx) return;
  ctx.value = _ctx;

  const _magnifier = magnifier.value;
  if (!_magnifier) return;
  magnifier.value = _magnifier;

  var img = new Image();
  img.onload = function () {
    ctx.value?.drawImage(img, 0, 0);
    refreshPaletteData();
  };
  img.src = "scenery.jpeg";
});

function onFileChange(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (ctx.value && file) {
    const img = new Image();
    img.onload = () => {
      if (imageCanvas.value) {
        imageCanvas.value.width = img.width;
        imageCanvas.value.height = img.height;
      }
      ctx.value?.drawImage(img, 0, 0);
      refreshPaletteData();
    };
    img.src = URL.createObjectURL(file);
  }
}

function handleColorBlk(idx: number) {
  switch(state.value){
    case 'normal':
      selectColorBlk(idx);
      break;
    case 'remove':
      paletteData.value.splice(idx, 1);
      break;
  }
}

function selectColorBlk(idx: number) {
  if (idx == selectedN.value) selectedN.value = -1;
  else selectedN.value = idx;
  refreshCanvas();
}

function paintFromArray(arr: Uint8ClampedArray) {
  if (!ctx.value || !imageSrcArray || !imageCanvas.value) return;
  ctx.value.putImageData(
    new ImageData(arr, imageCanvas.value.width, imageCanvas.value.height),
    0,
    0
  );
}

function refreshCanvas() {
  if (!ctx.value || !imageSrcArray || !imageCanvas.value) return;
  if (selectedN.value == -1) {
    paintFromArray(imageSrcArray);
    return;
  }
  let modImageData = new Uint8ClampedArray(
    imageCanvas.value.width * imageCanvas.value.height * 4
  );
  const epsilon = 50;
  const r_t = paletteData.value[selectedN.value].r;
  const g_t = paletteData.value[selectedN.value].g;
  const b_t = paletteData.value[selectedN.value].b;
  const a_t = paletteData.value[selectedN.value].a;
  for (let i = 0; i < imageSrcArray.length; i += 4) {
    const r = imageSrcArray[i];
    const g = imageSrcArray[i + 1];
    const b = imageSrcArray[i + 2];
    if((r_t - r)**2 + (g_t - g)**2 + (b_t - b)**2 > epsilon){
      const grayScale = getGrayScale({ r, g, b, a: 255 });
      modImageData[i] = modImageData[i + 1] = modImageData[i + 2] = grayScale;
      modImageData[i + 3] = imageSrcArray[i + 3] * 0.5;
    }else{
      modImageData[i] = r_t;
      modImageData[i + 1] = g_t;
      modImageData[i + 2] = b_t;
      modImageData[i + 3] = a_t;
    }
  }
  paintFromArray(modImageData);
}

function updateMagnifier(e: MouseEvent) {
  if (!magnifier.value || !ctx.value) return;
  const x = e.offsetX;
  const y = e.offsetY;
  magnifier.value.style.left = `${x + 30}px`;
  magnifier.value.style.top = `${y + 30}px`;
  magnifier.value.style.display = "block";

  const imgData = ctx.value.getImageData(x - 5, y - 5, 11, 11).data;
  selectedColor.value = {
    r: imgData[0 + (5 * 11 + 5) * 4],
    g: imgData[1 + (5 * 11 + 5) * 4],
    b: imgData[2 + (5 * 11 + 5) * 4],
    a: imgData[3 + (5 * 11 + 5) * 4],
  };
  for (let i = 0; i < 11; ++i) {
    for (let j = 0; j < 11; ++j) {
      if (imgData) {
        magnifierData.value[i][j] = {
          r: imgData[0 + (i * 11 + j) * 4],
          g: imgData[1 + (i * 11 + j) * 4],
          b: imgData[2 + (i * 11 + j) * 4],
          a: imgData[3 + (i * 11 + j) * 4],
        };
      }
    }
  }

  const focusBorderColor = isBright(selectedColor.value) ? "black" : "white";

  let targetElement = magnifier.value.querySelector<HTMLElement>(
    "tr:nth-child(5) td:nth-child(6)"
  );
  if (!targetElement) return;
  targetElement.style.borderBottomColor = focusBorderColor;
  targetElement = magnifier.value.querySelector<HTMLElement>(
    "tr:nth-child(6) td:nth-child(6)"
  );
  if (!targetElement) return;
  targetElement.style.borderColor = focusBorderColor;
  targetElement = magnifier.value.querySelector<HTMLElement>(
    "tr:nth-child(6) td:nth-child(5)"
  );
  if (!targetElement) return;
  targetElement.style.borderRightColor = focusBorderColor;
}

function hideMagnifier() {
  if (!magnifier.value) return;
  magnifier.value.style.display = "none";
}

function clickMagnifier() {
  if (state.value != "add") return;
  paletteData.value.push(magnifierData.value[5][5]);
}
</script>

<template>
  <div
    class="p-3 relative w-full sm:w-[640px] rounded-lg shadow-md bg-white border border-gray"
  >
    <input class="mb-2" type="file" accept="image/*" @change="onFileChange" />
    <div class="w-full h-full">
      <canvas
        class="block cursor-crosshair relative mx-auto"
        width="600"
        height="400"
        ref="imageCanvas"
        @mousemove="updateMagnifier"
        @mouseleave="hideMagnifier"
        @click="clickMagnifier"
      ></canvas>
      <div
        style="left: 0px; top: 0px"
        ref="magnifier"
        class="magnifier hidden z-50 overflow-hidden absolute rounded-full border border-solid border-black"
      >
        <table class="w-full h-full m-0 p-0 border-spacing-0">
          <tbody>
            <tr v-for="row in magnifierData">
              <td
                v-for="cell in row"
                :style="{
                  backgroundColor: `rgba(${cell.r}, ${cell.g}, ${cell.b}, ${
                    cell.a / 255.0
                  })`,
                }"
                class="border border-solid border-gray-400 p-0 box-border"
              />
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div
      class="mt-2 p-2 rounded-lg border border-gray-300 shadow-sm h-full w-full"
      id="palette"
    >
      <h1 class="font-bold text-sm uppercase">Palette</h1>
      <div class="flex gap-2 h-12">
        <div class="flex flex-col">
          <button class="PaletteBtn transition-colors" @click="handleAdd" :style="{ backgroundColor: state == 'add' ? 'aqua' : 'white' }">
            <svg
              width="15"
              height="15"
              viewBox="0 0 15 15"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
              class=""
            >
              <path
                d="M8 2.75C8 2.47386 7.77614 2.25 7.5 2.25C7.22386 2.25 7 2.47386 7 2.75V7H2.75C2.47386 7 2.25 7.22386 2.25 7.5C2.25 7.77614 2.47386 8 2.75 8H7V12.25C7 12.5261 7.22386 12.75 7.5 12.75C7.77614 12.75 8 12.5261 8 12.25V8H12.25C12.5261 8 12.75 7.77614 12.75 7.5C12.75 7.22386 12.5261 7 12.25 7H8V2.75Z"
                fill="currentColor"
                fill-rule="evenodd"
                clip-rule="evenodd"
              ></path>
            </svg>
          </button>
          <button class="PaletteBtn transition-colors" @click="handleRemove" :style="{ backgroundColor: state == 'remove' ? 'aqua' : 'white' }">
            <svg
              width="15"
              height="15"
              viewBox="0 0 15 15"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
              class=""
            >
              <path
                d="M2.25 7.5C2.25 7.22386 2.47386 7 2.75 7H12.25C12.5261 7 12.75 7.22386 12.75 7.5C12.75 7.77614 12.5261 8 12.25 8H2.75C2.47386 8 2.25 7.77614 2.25 7.5Z"
                fill="currentColor"
                fill-rule="evenodd"
                clip-rule="evenodd"
              ></path>
            </svg>
          </button>
        </div>
        <div class="palette w-full flex rounded-lg overflow-hidden">
          <div
            v-for="(color, idx) of paletteData"
            class="flex-1 cursor-pointer flex"
            :style="{
              backgroundColor: `rgba(${color.r}, ${color.g}, ${color.b}, ${
                color.a / 255.0
              })`,
            }"
            @click="handleColorBlk(idx)"
          >
            <div
              v-if="selectedN == idx"
              class="h-2.5 w-2.5 mx-auto my-auto rounded-full"
              :style="{
                backgroundColor: isBright(color) ? 'black' : 'white',
              }"
            ></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
div.magnifier {
  width: 110px;
  height: 110px;
  background-size: 6px;
  background-color: white;
  background-image: url("data:image/gif;base64,R0lGODlhDAAMAIABAMzMzP///yH5BAEAAAEALAAAAAAMAAwAAAIWhB+ph5ps3IMyQFBvzVRq3zmfGC5QAQA7");
}
div.palette {
  background-size: 6px;
  background-color: white;
  background-image: url("data:image/gif;base64,R0lGODlhDAAMAIABAMzMzP///yH5BAEAAAEALAAAAAAMAAwAAAIWhB+ph5ps3IMyQFBvzVRq3zmfGC5QAQA7");
}
div.magnifier td {
  width: calc(100% / 11);
  height: calc(100% / 11);
}
div.magnifier tr:nth-child(5) td:nth-child(6) {
  border-bottom-color: rgb(90, 90, 90);
}
div.magnifier tr:nth-child(6) td:nth-child(5) {
  border-right-color: rgb(90, 90, 90);
}
div.magnifier tr:nth-child(6) td:nth-child(6) {
  border-color: rgb(90, 90, 90);
}
div#palette button.PaletteBtn {
  border-width: 1px;
  padding-left: 0.75rem;
  padding-right: 0.75rem;
  padding-top: 0.25rem;
  padding-bottom: 0.25rem;
  border-radius: 0.375rem;
  display: flex;
  justify-content: center;
  --tw-border-opacity: 1;
  border-color: rgb(209 213 219 / var(--tw-border-opacity)) /* #d1d5db */;
}
div#palette button.PaletteBtn:hover {
  --tw-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --tw-shadow-colored: 0 4px 6px -1px var(--tw-shadow-color),
    0 2px 4px -2px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000),
    var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow);
  --tw-border-opacity: 1;
  border-color: rgb(156 163 175 / var(--tw-border-opacity)) /* #9ca3af */;
}
</style>
