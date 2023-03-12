<template>
  <div class="container">
    <div class="display-container">
      <div v-if="!imageLoading" class="image-container">
        <div v-for="url in imageUrls" :key="url" class="image-holder">
          <img :src="url" />
        </div>
      </div>
      <div
        v-else
        class="flex items-center justify-center full-width full-height"
      >
        <q-spinner-grid color="teal" size="72px" />
      </div>
    </div>
    <div class="prompt-holder">
      <q-input
        class="prompt-input"
        v-model="defaultPrompt"
        placeholder="Enter a prompt..."
        @keyup.enter="submitPrompt"
        outlined
        clearable
        autogrow
      />
      <q-btn
        class="q-ml-sm"
        color="teal-4"
        label="Generate"
        unelevated
        @click="submitPrompt"
      />
    </div>
    <div class="art-styles-holder">
      <q-checkbox
        v-for="style in artStyles"
        :key="style"
        v-model="artStylesModel"
        :val="style"
        keep-color
        color="teal"
        checked-icon="star"
        unchecked-icon="star_border"
        :label="style"
        label-position="right"
        dense
      />
    </div>
  </div>
</template>

<script>
import { Configuration, OpenAIApi } from "openai";
const configuration = new Configuration({
  apiKey: process.env.OPEN_AI_KEY,
});

export default {
  name: "ImageCreator",
  data() {
    return {
      defaultPrompt: "",
      imageUrl: "",
      imageUrls: [],
      imageLoading: false,
      artStyles: [
        "3D",
        "abstract",
        "anime",
        "cartoon",
        "comic book",
        "cyber punk",
        "digital art",
        "fantasy",
        "graffiti",
        "hand drawn sketch",
        "hyperrealist",
        "low poly",
        "matisse",
        "minimalist",
        "painting",
        "pixel",
        "pop art",
        "portrait",
        "renaissance",
        "synthwave",
        "watercolor",
        "vaporwave",
      ].map((style) =>
        style
          .split(" ")
          .map((word) => word[0].toUpperCase() + word.slice(1))
          .join(" ")
      ),
      artStylesModel: [],
    };
  },
  computed: {
    prompt() {
      let prompt = this.defaultPrompt;
      if (this.artStylesModel.length > 0) {
        prompt += ` ${this.artStylesModel.join(", ")}`;
      }
      return prompt;
    },
  },
  methods: {
    async submitPrompt() {
      this.imageLoading = true;
      try {
        const openai = new OpenAIApi(configuration);
        const response = await openai.createImage({
          prompt: this.prompt,
          n: 4,
          size: "256x256",
        });

        this.imageUrls = response.data.data.map((item) => item.url);

        console.log(response);
        this.imageLoading = false;
      } catch (error) {
        console.log(error);
        this.imageLoading = false;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: calc(100vh - 50px);
  padding: 48px;
}

.display-container {
  width: 100%;
  height: 85%;
  max-height: 85%;
  margin-bottom: 24px;
  border-radius: 8px;
  overflow: hidden;

  .image-container {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    grid-template-rows: repeat(2, minmax(0, 1fr));
    grid-gap: 4px;
    height: 100%;
    max-height: 100%;
  }

  .image-holder {
    width: 100%;
    height: 100%;

    img {
      object-fit: cover;
      height: 100%;
      max-height: 100%;
      width: 100%;
      max-width: 100%;
      border-radius: 8px;
    }
  }
}

.prompt-holder {
  width: 100%;
  display: flex;
  flex-direction: row;
  gap: 16;

  .prompt-input {
    flex: 1;
  }
}

.art-styles-holder {
  width: 100%;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 16px;
  background: #f6f5f5;
  padding: 8px;
  border-radius: 8px;
}
</style>