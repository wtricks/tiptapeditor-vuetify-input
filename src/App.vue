<template>

  <h1>As a text-field</h1>
  <Editor 
    size="textfield"
    label="Your thoughts" 
    v-model="content" 
  />

  <h1>As a text-area</h1>
  <Editor 
    size="textarea"
    label="Write your message" 
    v-model="content" 
  />

  <h1>Required field</h1>
  <Editor  
    required
    label="Write your message" 
    v-model="content" 
  />

  <h1>Custom Required message</h1>
  <Editor  
    required
    label="Write your message" 
    v-model="content" 
    required-message="This field is required. please fill it"
  />

  <h1>Auto grow field</h1>
  <Editor
    auto-grow
    size="textarea"
    label="Write your message" 
    v-model="content" 
  />

  <h1>With Hint</h1>
  <Editor
    hint="A message that is need to be sent."
    label="Write your message" 
    v-model="content" 
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import Editor from "./components/CustomEditor/index.vue";

const content = ref('<b>A normal message here...</b>')
</script>

<style>
body {
  margin: 40px;
}

h1 {
  font-size: 18px;
  margin-bottom: 10px;
  margin-top: 1rem;
  text-decoration: underline;
}
</style>
