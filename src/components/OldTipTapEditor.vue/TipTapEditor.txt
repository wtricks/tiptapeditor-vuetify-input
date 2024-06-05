<script lang="ts" setup>
import {onMounted, onBeforeUnmount} from 'vue'
import {Editor, EditorContent, JSONContent} from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Mention, {MentionOptions} from '@tiptap/extension-mention'
import {VueRenderer} from '@tiptap/vue-3'
import MentionList from './MentionList.vue'
import tippy from 'tippy.js'
import axios from "@axios";
import {usePage} from "@inertiajs/vue3";

const props = defineProps({
  content: String,
})

const editor = shallowRef<Editor | null>(null)
const notebookUUID = shallowRef<String>(usePage().props.notebook?.uuid ?? '')

const suggestion = {
  items: async ({query}) => {

    if (notebookUUID.value) {
      // Make an API call to fetch mention suggestions
      const data = (await axios(route('notebooks.items', {
        notebook: notebookUUID.value ?? '',
        query: query
      }))).data

      if (data) {
        // Extract suggestion titles and ids from the API response
        const suggestions = data.map(item => ({id: item.id, title: item.title, type: item.type}))

        // Filter suggestions based on the query and limit to 5 items
        return suggestions.filter(item => item.title.toLowerCase().startsWith(query.toLowerCase())).slice(0, 5)
      }
    }

    return [];
  },

  render: () => {
    let component
    let popup

    return {
      onStart: props => {
        component = new VueRenderer(MentionList, {
          props,
          editor: props.editor,
        })

        if (!props.clientRect) {
          return
        }

        popup = tippy('body', {
          getReferenceClientRect: props.clientRect,
          appendTo: () => document.body,
          content: component.element,
          showOnCreate: true,
          interactive: true,
          trigger: 'manual',
          placement: 'bottom-start',
        })
      },

      onUpdate(props) {
        component.updateProps(props)

        if (!props.clientRect) {
          return
        }

        popup[0].setProps({
          getReferenceClientRect: props.clientRect,
        })
      },

      onKeyDown(props) {
        if (props.event.key === 'Escape') {
          popup[0].hide()
          return true
        }

        if (props.event.key === 'Enter') {
          popup[0].hide()
          return true
        }

        if (typeof component.ref?.onKeyDown === 'function') {
          return component.ref?.onKeyDown(props)
        }
      },

      onExit() {
        popup[0].destroy()
        component.destroy()
      },
    }
  },
}

onMounted(() => {
  editor.value = new Editor({
    extensions: [
      StarterKit,
      Mention.configure({
        HTMLAttributes: {
          class: 'mention',
        },
        suggestion,
        renderHTML({node, HTMLAttributes}: { node: JSONContent, HTMLAttributes: Record<string, any> }) {
          const {id, label} = node.attrs

          return [
            'span',
            {
              ...HTMLAttributes,
              'data-id': id,
              class: 'mention',
            },
            `@${label}`,
          ]
        },
      } as MentionOptions),
    ],
    content: props.content,
  })
})


onBeforeUnmount(() => {
  if (editor.value) {
    editor.value.destroy()
  }
})

</script>

<template>
  <template v-if="editor">
    <div
        class="v-input v-input--horizontal v-text-field">
      <div class="v-input__control ">
        <div
            class="v-field v-field--variant-outlined">
          <div class="v-field__overlay"></div>
          <div class="v-field__field">
            <label class="v-label v-field-label" for="input-26" style="">Idea</label>
            <EditorContent class="v-field__input" v-bind="$attrs" id="input-26" :editor="editor"/>
          </div>
          <div class="v-field__outline">
            <div class="v-field__outline__start"></div>
            <div class="v-field__outline__notch">
              <label class="v-label v-field-label v-field-label--floating"
                     aria-hidden="true" for="input-62" style="">Title</label>
            </div>
            <div class="v-field__outline__end"></div></div>
        </div>
      </div>
      <div class="v-input__details">
        <div class="v-messages" role="alert" aria-live="polite" id="input-62-messages">
          <div class="v-messages__message"></div>
        </div><!----></div>
    </div>
  </template>
</template>

<style lang="scss">
.tiptap {
  width: 100%;

  > * + * {
    margin-top: 0.75em;
  }
}

.ProseMirror {
  border: 1px solid grey;
  border-radius: 4px;
  padding: 10px;
}

.mention {
  border: 1px solid #000;
  border-radius: 0.4rem;
  padding: 0.1rem 0.3rem;
  box-decoration-break: clone;
}
</style>
