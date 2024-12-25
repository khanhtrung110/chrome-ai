<script setup>
import { ref, onMounted } from "vue";
const messageDefault =
  "In the 21st century, technology has become an indispensable part of daily life. The internet connects people worldwide, fostering e-commerce and online education. However, over-reliance on technology has also led to challenges, such as a lack of work-life balance, reduced quality of face-to-face communication, and increased risks to personal data security. Therefore, using technology responsibly and efficiently is crucial to harness its benefits while avoiding negative consequences.";
const message = ref(
  "In the 21st century, technology has become an indispensable part of daily life. The internet connects people worldwide, fostering e-commerce and online education. However, over-reliance on technology has also led to challenges, such as a lack of work-life balance, reduced quality of face-to-face communication, and increased risks to personal data security. Therefore, using technology responsibly and efficiently is crucial to harness its benefits while avoiding negative consequences."
);

let summarizer;
let translator;
const isSummarizer = ref(false);
const isTranslator = ref(false);
const isLoading = ref(false);

const createSummarizationSession = async () => {
  const options = {
    sharedContext: "This is a scientific article",
    type: "tl;dr",
    format: "plain-text",
    length: "short",
  };

  const available = (await self.ai.summarizer.capabilities()).available;
  if (available === "no") {
    // The Summarizer API isn't usable.
    return;
  }
  if (available === "readily") {
    // The Summarizer API can be used immediately .
    summarizer = await self.ai.summarizer.create(options);
  } else {
    // The Summarizer API can be used after the model is downloaded.
    summarizer = await self.ai.summarizer.create(options);
    summarizer.addEventListener("downloadprogress", (e) => {
      console.log(e.loaded, e.total);
    });
    await summarizer.ready;
  }
};

const createTranslatorSession = async () => {
  translator = await self.translation.createTranslator({
    sourceLanguage: "en",
    targetLanguage: "vi",
  });
};

const handleSummarizer = async () => {
  isSummarizer.value = !isSummarizer.value;
  if (isSummarizer.value) {
    isLoading.value = true;
    message.value = await summarizer.summarize(messageDefault, {
      context: "This article is intended for a tech-savvy audience.",
    });
    isLoading.value = false;
  } else {
    message.value = messageDefault;
  }
};

const handleTranslate = async () => {
  isTranslator.value = !isTranslator.value;
  if (isTranslator.value) {
    isLoading.value = true;
    message.value = await translator.translate(messageDefault);
    isLoading.value = false;
  } else {
    message.value = messageDefault;
  }
};

onMounted(() => {
  createSummarizationSession();
  createTranslatorSession();
});
</script>

<template>
  <div class="ai-chrome" :style="isLoading ? 'opacity: 0.8' : ''">
    <div class="message">
      {{ message }}
    </div>
    <div class="action">
      <div @click="handleSummarizer" class="action-item">
        <i
          class="mdi mdi-arrow-expand-all"
          style="font-size: 20px; color: #3076a5"
        ></i>
      </div>
      <div @click="handleTranslate" class="action-item">
        <i
          class="mdi mdi-translate"
          style="font-size: 20px; color: #3076a5"
        ></i>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ai-chrome {
  position: relative;
}
.message {
  padding: 12px;
  border-radius: 6px;
  background: #f5f6f6;
  color: #013253;
}
.action {
  padding: 2px;
  box-shadow: 0px 0px 1px 0px #b5b5b51a, 0px 2px 2px 0px #b5b5b517,
    0px 3px 2px 0px #b5b5b50d, 0px 6px 2px 0px #b5b5b503,
    0px 9px 3px 0px #b5b5b500;
  border-radius: 4px;
  background: white;
  position: absolute;
  right: 5px;
  top: 5px;
  display: none;
  gap: 4px;
}
.ai-chrome:hover .action {
  display: flex;
}

.action-item {
  border-radius: 2px;
  width: 24px;
  height: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}
.action-item:hover {
  background: #f5f6f6;
}
</style>
