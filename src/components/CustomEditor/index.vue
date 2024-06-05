<template>
  <v-text-field
    :model-value="textFieldContent"
    :label="label"
    :required="required"
    :rules="validationRules"
    outlined
    v-bind="$attrs"
    v-on="listeners"
    @click:control="focusEditor"
    :class="{[randomId]: true, 'is-textarea': size == 'textarea', 'auto-grow': autoGrow }"
    :style="{'--min-height': `${minHeight||90}px`, '--max-height': `${maxHeight||90}px`}"
  >
    <template #default>
      <EditorContent
        v-if="editor"
        :editor="editor"
        class="editor-content"
        @mousedown.stop
        @focus="focusEditor"
      />
    </template>
  </v-text-field>
</template>

<script lang="ts" setup>
import {
  ref,
  computed,
  onMounted,
  onBeforeUnmount,
  shallowRef,
  watch,
  nextTick
} from "vue";
import { Editor, EditorContent, JSONContent, VueRenderer } from "@tiptap/vue-3";
import Mention, { MentionOptions } from "@tiptap/extension-mention";
import MentionList from "./MentionList.vue";
import StarterKit from "@tiptap/starter-kit";
import { Extension } from "@tiptap/core";
import tippy from "tippy.js";
// import axios from "@axios";
// import {usePage} from "@inertiajs/vue3";

interface Props {
  label: string;
  required?: boolean;
  rules?: Array<(value: string) => boolean | string>;
  size?: "textfield" | "textarea";
  minHeight?: number;
  maxHeight?: number;
  autoGrow?: boolean;
  requiredMessage?: string;
}

const props = defineProps<Props>();
// const notebookUUID = shallowRef<String>(usePage().props.notebook?.uuid ?? '')

const editor = shallowRef<Editor | null>(null);
const content = defineModel<string>();
const isFocused = ref(false);
const randomId = `input-${Math.random().toString(36).substr(2, 9)}`;
const listeners = computed(() => {
  return {
    ...Object.keys(props).reduce((listeners, key) => {
      if (key.startsWith("on")) {
        listeners[key] = (props as any)[key];
      }
      return listeners;
    }, {} as Record<string, any>),
  };
});

const focusEditor = (event?: MouseEvent) => {
  if (editor.value) {
    if (event) {
      event.preventDefault();
    }

    nextTick(() => {
      editor.value?.commands.focus();
      isFocused.value = true;

      if (event) {
        const pos = editor.value!.view.posAtCoords({
          left: event.clientX,
          top: event.clientY,
        });

        if (pos) {
          editor.value!.chain().setTextSelection(pos.pos).run();
        }
      }
    });
  }
};

const suggestion = {
  items: async ({ query }: { query: string }) => {
    /*
    if (notebookUUID.value) {
      // Make an API call to fetch mention suggestions
      const data = (
        await axios(
          route("notebooks.items", {
            notebook: notebookUUID.value ?? "",
            query: query,
          })
        )
      ).data;

      if (data) {
        // Extract suggestion titles and ids from the API response
        const suggestions = data.map((item) => ({
          id: item.id,
          title: item.title,
          type: item.type,
        }));

        // Filter suggestions based on the query and limit to 5 items
        return suggestions
          .filter((item) =>
            item.title.toLowerCase().startsWith(query.toLowerCase())
          )
          .slice(0, 5);
      }
    } 
    */

    console.log(query)

    // used dummy for testing perpose only
    const dummyData = [
      { id: 1, title: "John Doe", type: "user" },
      { id: 2, title: "Jane Smith", type: "user" },
      { id: 3, title: "Alice Johnson", type: "user" },
      { id: 4, title: "Bob Brown", type: "user" },
      { id: 5, title: "Charlie Davis", type: "user" },
      { id: 6, title: "Diana Evans", type: "user" },
      { id: 7, title: "Edward Foster", type: "user" },
      { id: 8, title: "Fiona Green", type: "user" },
      { id: 9, title: "George Harris", type: "user" },
      { id: 10, title: "Hannah Irvine", type: "user" }
    ];

    // used dummy for testing perpose only
    return dummyData;

    // return [];
  },
  render: () => {
    let component: VueRenderer | undefined;
    let popup: ReturnType<typeof tippy> | undefined;

    return {
      onStart: (props: any) => {
        component = new VueRenderer(MentionList, {
          props,
          editor: props.editor,
        });

        if (!props.clientRect) {
          return;
        }

        popup = tippy("body", {
          getReferenceClientRect: props.clientRect,
          appendTo: () => document.body,
          content: component.element,
          showOnCreate: true,
          interactive: true,
          trigger: "manual",
          placement: "bottom-start",
        });
      },
      onUpdate(props: any) {
        component?.updateProps(props);
        if (!props.clientRect) {
          return;
        }
        popup?.[0].setProps({
          getReferenceClientRect: props.clientRect,
        });
      },
      onKeyDown(props: any) {
        if (props.event.key === "Escape") {
          popup?.[0].hide();
          return true;
        }
        if (props.event.key === "Enter") {
          popup?.[0].hide();
          return true;
        }
        if (typeof component?.ref?.onKeyDown === "function") {
          return component.ref?.onKeyDown(props);
        }
      },
      onExit() {
        popup?.[0].destroy();
        component?.destroy();
      },
    };
  },
};

const textFieldContent = computed(() => {
  return content.value!.replace(/<[^>]*>/g, "").trim();
});

const validationRules = computed(() => {
  const rules = props.rules ? [...props.rules] : [];
  if (props.required && !props.rules) {
    rules.push((value: string) => !!value || props.requiredMessage || "This field is required.");
  }
  return rules;
});

// custom extension to handle the Enter key
const CustomEnterHandler = Extension.create({
  addKeyboardShortcuts() {
    return {
      Enter: () => {
        if (!props.size || props.size === 'textfield') {
          return true; // Prevent default Enter behavior
        }
        return false;
      }
    }
  }
});

watch(isFocused, (value: boolean) => {
  const input = document.querySelector(
    "." + randomId + " .v-field__input input"
  ) as HTMLInputElement;
  
  input.style.display = "block";
  !value && input.focus();

  nextTick(() => {
    input.blur();
    input.style.display = "none";
  });
});

watch(content, (newValue) => {
  if (editor.value && newValue !== editor.value.getHTML()) {
    editor.value.commands.setContent(newValue as string, false);
  }
});

onMounted(() => {
  editor.value = new Editor({
    extensions: [
      StarterKit,
      // @ts-ignore
      Mention.configure({
        HTMLAttributes: {
          class: "mention",
        },
        suggestion,
        renderHTML({
          node,
          HTMLAttributes,
        }: {
          node: JSONContent;
          HTMLAttributes: Record<string, any>;
        }) {
          // @ts-ignore
          const { id, label } = node.attrs;
          return [
            "span",
            {
              ...HTMLAttributes,
              "data-id": id,
              class: "mention",
            },
            `@${label}`,
          ];
        },
      } as MentionOptions),
      CustomEnterHandler
    ],
    content: content.value,
    onUpdate({ editor }) {
      content.value = editor.getHTML();
    },
    onFocus() {
      isFocused.value = true;
    },
    onBlur() {
      isFocused.value = false;
    }
  });
});

onBeforeUnmount(() => {
  if (editor.value) {
    editor.value.destroy();
  }
});
</script>

<style lang="scss">
.editor-content {
  .ProseMirror {
    outline: none;
  }
  .ProseMirror p {
    margin: 0;
  }
  .mention {
    background-color: #e0e0e0;
    border-radius: 4px;
    padding: 0 4px;
  }
}

.v-input:not(.is-textarea) {
  .editor-content .ProseMirror {
    overflow-x: auto;
    p,
    h1,
    h2,
    h3,
    h4,
    h5,
    h6,
    div,
    ul,
    li {
      display: inline-block;
      margin-right: 5px;
    }
  }
  .editor-content .ProseMirror::-webkit-scrollbar {
    display: none;
  }
}

.v-text-field.is-textarea {
  .editor-content .ProseMirror {
    min-height: var(--min-height);
    max-height: var(--max-height);
    overflow-y: auto;
  }

  &.auto-grow .editor-content .ProseMirror {
    max-height: unset;
  }

  .v-label.v-field-label:not(.v-field-label--floating) {
    top: 18%;
  }
}

.editor-content {
  width: 100%;
}

.v-field__input input {
  display: none;
  position: absolute;
  opacity: 0;
}

.editor-content .ProseMirror {
  white-space: pre;
}
</style>
