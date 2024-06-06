<template>

  <h1>As a text-field</h1>
  <Editor 
    size="textfield"
    label="Your thoughts" 
    v-model="content1" 
  />

  <h1>As a text-area</h1>
  <Editor 
    size="textarea"
    label="Write your message" 
    v-model="content2" 
  />

  <h1>Required field</h1>
  <Editor  
    required
    label="Write your message" 
    v-model="content3" 
  />

  <h1>Custom Required message</h1>
  <Editor  
    required
    label="Write your message" 
    v-model="content4" 
    required-message="This field is required. please fill it"
  />

  <h1>Auto grow field</h1>
  <Editor
    auto-grow
    size="textarea"
    label="Write your message" 
    v-model="content5" 
  />

  <h1>With Hint</h1>
  <Editor
    hint="A message that is need to be sent."
    label="Write your message" 
    v-model="content6" 
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import Editor from "./components/CustomEditor/index.vue";

const content1 = ref('<b style="color:blue">A normal message here...</b><h2>A heading content</h2>')
const content2 = ref('<b style="color:blue">A normal message here...</b><h2>A heading content</h2>')
const content3 = ref('<b style="color:blue">A normal message here...</b><h2>A heading content</h2>')
const content4 = ref('<b style="color:blue">A normal message here...</b><h2>A heading content</h2>')
const content5 = ref('<b style="color:blue">A normal message here...</b><h2>A heading content</h2>')
const content6 = ref('<b style="color:blue">A normal message here...</b><h2>A heading content</h2>')
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
