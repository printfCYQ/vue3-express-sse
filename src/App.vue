<template>
  <div>
    {{ chatText }}
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
const chatText = ref('')

onMounted(() => {
  getRes();
})

const getRes = async () => {
  try {
    const res = await fetch("http://localhost:3000/sse", {
      method: "get",
    });
    const reader = res.body?.getReader();
    let text = "";
    while (reader) {
      const { value, done } = await reader.read();
      const chars = new TextDecoder().decode(value);
      if (done) {
        break;
      }
      const dataArray = chars.trim().split("\n\n");
      const jsonObjects = dataArray.map((data) => {
        const jsonString = data.substring("data: ".length);
        return JSON.parse(jsonString);
      });
      jsonObjects.forEach((item) => {
        text += item.content;
      });
      chatText.value = text;
    }
  } catch (error) {
    console.log("error", error);
  }
};
</script>

<style scoped></style>