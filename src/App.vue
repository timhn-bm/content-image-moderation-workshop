<script setup lang="ts">
import { computed, ref } from "vue";

const imageUrl = ref("");

type ApiErrorResponse = {
  detail: {
    detail: { face_detected: "yes" | "no"; rejected: "yes" | "no" };
    explanation: string;
    reason: string[];
  };
};
const apiResponse = ref<string | ApiErrorResponse>();

const imageAccepted = computed(() => apiResponse.value === "OK");
const imageRefused = computed(
  () => !!apiResponse.value && typeof apiResponse.value === "object"
);

const faceDetected = computed(
  () =>
    typeof apiResponse.value === "object" &&
    apiResponse.value.detail.detail.face_detected === "yes"
);
const rejected = computed(
  () =>
    typeof apiResponse.value === "object" &&
    apiResponse.value.detail.detail.rejected === "yes"
);

const isFetching = ref(false);

async function handleClick() {
  isFetching.value = true;
  apiResponse.value = undefined;

  try {
    const baseURL = "https://abc0-176-162-136-18.ngrok-free.app/review-image";
    const response = await fetch(`${baseURL}?image_url=${imageUrl.value}`, {
      headers: {
        "ngrok-skip-browser-warning": "true",
      },
    });

    apiResponse.value = await response.json();

    console.log(apiResponse.value);
    isFetching.value = false;
  } catch {
    isFetching.value = false;
  }
}
</script>

<template>
  <div class="container">
    <h1>AI Image Moderation</h1>

    <div class="label-left">
      <label for="input">Enter image url</label>

      <input
        class="input__field"
        type="text"
        placeholder="Enter an image URL"
        v-model="imageUrl"
        name="input"
      />
    </div>

    <img class="image" :src="imageUrl" v-if="imageUrl" />

    <button
      :disabled="!imageUrl || isFetching"
      class="button"
      role="button"
      @click="handleClick"
    >
      Analyze my image
    </button>

    <div v-if="!isFetching && imageAccepted">
      <div style="width: fit-content; margin: auto" class="badge-success">
        Valid
      </div>
    </div>

    <div v-if="!isFetching && imageRefused && typeof apiResponse === 'object'">
      <div style="display: flex; justify-content: center; gap: 20px">
        <div style="display: flex; align-items: center; gap: 20px">
          <h3>Face detected</h3>

          <div :class="faceDetected ? 'badge-error' : 'badge-success'">
            {{ faceDetected ? "Yes" : "No" }}
          </div>
        </div>

        <div style="display: flex; align-items: center; gap: 20px">
          <h3>Image rejected</h3>

          <div :class="rejected ? 'badge-error' : 'badge-success'">
            {{ rejected ? "Yes" : "No" }}
          </div>
        </div>
      </div>

      <h3>Explanation</h3>

      <p style="width: 100%; font-size: 16px" rows="5">
        {{ apiResponse.detail.explanation }}
      </p>
    </div>
  </div>
</template>

<style scoped>
:root {
  --color-light: white;
  --color-dark: #212121;
  --color-signal: #fab700;

  --color-background: var(--color-light);
  --color-text: var(--color-dark);
  --color-accent: var(--color-signal);

  --size-bezel: 0.5rem;
  --size-radius: 4px;

  line-height: 1.4;

  font-family: "Inter", sans-serif;
  font-size: calc(0.6rem + 0.4vw);
  color: var(--color-text);
  background: var(--color-background);
  font-weight: 300;
  padding: 0 calc(var(--size-bezel) * 3);
}

/* CONTAINER */
.container {
  width: 800px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  gap: 20px;
  margin-bottom: 24px;
}

/* LABEL LEFT */
.label-left {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

/* IMAGE */
.image {
  max-height: 400px;
  margin: auto;
  outline: 5px solid pink;
  outline-offset: 5px;
}

/* INPUT */
.input {
  position: relative;
}

.input__label {
  position: absolute;
  left: 0;
  top: 0;
  padding: calc(var(--size-bezel) * 0.75) calc(var(--size-bezel) * 0.5);
  margin: calc(var(--size-bezel) * 0.75 + 3px) calc(var(--size-bezel) * 0.5);
  background: pink;
  white-space: nowrap;
  transform: translate(0, 0);
  transform-origin: 0 0;
  background: var(--color-background);
  transition: transform 120ms ease-in;
  font-weight: bold;
  line-height: 1.2;
}

.input__field {
  box-sizing: border-box;
  display: block;
  width: 100%;
  border: 3px solid currentColor;
  padding: calc(var(--size-bezel) * 1.5) var(--size-bezel);
  color: currentColor;
  background: transparent;
  border-radius: var(--size-radius);
  padding: 8px;

  &:focus,
  &:not(:placeholder-shown) {
    & + .input__label {
      transform: translate(0.25rem, -65%) scale(0.8);
      color: var(--color-accent);
    }
  }
}

/* BUTTON */
.button {
  appearance: none;
  background-color: greenyellow;
  border: 1px solid rgba(27, 31, 35, 0.15);
  box-shadow: rgba(27, 31, 35, 0.04) 0 1px 0,
    rgba(255, 255, 255, 0.25) 0 1px 0 inset;
  box-sizing: border-box;
  color: #24292e;
  cursor: pointer;
  display: inline-block;
  font-family: -apple-system, system-ui, "Segoe UI", Helvetica, Arial,
    sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  font-size: 16px;
  font-weight: 500;
  line-height: 20px;
  list-style: none;
  padding: 6px 16px;
  position: relative;
  transition: background-color 0.2s cubic-bezier(0.3, 0, 0.5, 1);
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  margin: auto;
}

.button:hover {
  background-color: green;
  color: #959da5;
  text-decoration: none;
  transition-duration: 0.1s;
}

.button:disabled {
  background-color: green;
  border-color: rgba(27, 31, 35, 0.15);
  color: #959da5;
  cursor: default;
}

.button:active {
  background-color: green;
  color: #959da5;
  box-shadow: rgba(225, 228, 232, 0.2) 0 1px 0 inset;
  transition: none 0s;
}

.button:focus {
  outline: 1px transparent;
}

.button:before {
  display: none;
}

.button:-webkit-details-marker {
  display: none;
}

/* BADGE */
.badge-error {
  background-color: hsla(301, 100%, 50%, 0.922);
  height: fit-content;
  font-size: bold;
  color: white;
  padding: 4px 8px;
  text-align: center;
  border-radius: 5px;
}

.badge-success {
  background-color: #17da78;
  color: darkred;
  height: fit-content;
  font-size: bold;
  color: white;
  padding: 4px 8px;
  text-align: center;
  border-radius: 5px;
}
</style>
