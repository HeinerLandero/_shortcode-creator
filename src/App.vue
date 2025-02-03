<script setup>
import { ref } from 'vue';
import JSZip from 'jszip';
import Footer from '@/components/Footer.vue';
import BottonDownLoad from './components/BottonDownLoad.vue';
import BottonAdd from './components/BottonAdd.vue';


const shortcodeName = ref('');

const jsonData = ref({
  caption: '',
  blocked: {
    DELETE: [],
    CLONE: [],
    MOVE: [],
    BLOCK: [],
  },
  groups: [
    {
      caption: '',
      comment: '',
      blocked: [],
      fields: [],
    },
  ],
  fields: [],
});

const shortcodes = ref([]);

const addField = () => {
  jsonData.value.groups[0].fields.push({
    caption: '',
    id: '',
    type: 'input',
    blocked: [],
    width: 6,
  });
};

const deleteField = (index) => {
  jsonData.value.groups[0].fields.splice(index, 1);
};

let draggedIndex = null;

const onDragStart = (index) => {
  draggedIndex = index;
};

const onDrop = (dropIndex) => {
  if (draggedIndex !== null && draggedIndex !== dropIndex) {
    const fields = jsonData.value.groups[0].fields;
    const [movedField] = fields.splice(draggedIndex, 1);
    fields.splice(dropIndex, 0, movedField);
  }
  draggedIndex = null;
};

const updateBlockedArray = (key, value) => {
  const newValues = value
    .split(',')
    .map((item) => parseFloat(item.trim()))
    .filter((item) => !isNaN(item)); 

  jsonData.value.blocked[key] = newValues;
};


const addShortcode = () => {
  if (!shortcodeName.value.trim()) {
    alert('Please provide a shortcode name.');
    return;
  }

  shortcodes.value.push({
    name: shortcodeName.value,
    data: jsonData.value,
  });

  shortcodeName.value = '';
  jsonData.value = {
    caption: '',
    blocked: {
      DELETE: [],
      CLONE: [],
      MOVE: [],
      BLOCK: [],
    },
    groups: [
      {
        caption: '',
        comment: '',
        blocked: [],
        fields: [],
      },
    ],
    fields: [],
  };
};

const downloadJsonFiles = () => {
  if (shortcodes.value.length === 0) {
    alert('No shortcodes to download.');
    return;
  }

  const zip = new JSZip();

  shortcodes.value.forEach((shortcode) => {
    const jsonBlob = new Blob([formatJson(shortcode.data)], {
      type: 'application/json',
    });
    zip.file(`${shortcode.name}.json`, jsonBlob);

    const phpContent = generatePhpFile(shortcode.data);
    const phpBlob = new Blob([phpContent], { type: 'application/php' });
    zip.file(`${shortcode.name}.php`, phpBlob);
  });

  zip.generateAsync({ type: 'blob' }).then((content) => {
    const link = document.createElement('a');
    link.href = URL.createObjectURL(content);
    link.download = 'shortcodes.zip';
    link.click();
  });
};

const generatePhpFile = (data) => {
  const fields = data.groups[0]?.fields || [];
  let phpCode = `<?php\n\n    $params = getFieldsShortcodes(json_decode($_GET['shortcode_json']));\n`;

  fields.forEach((field) => {
    const fieldId = field.id;
    phpCode += `    $${fieldId} = (isset($params->${fieldId})) && !empty($params->${fieldId}) ? $params->${fieldId} : null;\n`;
  });

  phpCode += '\n?>';
  return phpCode;
};

const formatJson = (json) => {
  return JSON.stringify(json, null, 2)
    .replace(/(\[\d+\])/g, (match) => match.replace(/ \n \s*/g, ''))
    .replace(/(\[\])/g, (match) => match.replace(/ \n \s*/g, ''))
    .replace(/(\[[\d, ]+\])/g, (match) => match.replace(/ \n \s*/g, ''));
};
</script>

<template>
  <h1>Shortcode Creator</h1>
  <div class="wrapper">
    <!-- JSON  Preview -->
    <main>
      <h2>JSON Preview</h2>
      <pre>{{ formatJson(jsonData) }}</pre>
    </main>
    <BottonAdd class="button_add_list" @click.prevent="addShortcode" >Add Shortcode</BottonAdd>

    <aside class="aside_tools">
      <div class="aside_shortcodes">
        <h2>Shortcodes List</h2>
        <ul>
          <li v-for="(shortcode, index) in shortcodes" :key="index">
            {{ shortcode.name }}
          </li>
        </ul>
        <BottonDownLoad @click.prevent="downloadJsonFiles" />
      </div>
      <h2>JSON Builder</h2>
      <form>
        <label for="shortcode-name">Shortcode Name:</label>
          <input
            id="shortcode-name"
            v-model="shortcodeName"
            type="text"
            placeholder="Enter shortcode name"
          />

        <label for="caption">Caption:</label>
        <input
          id="caption"
          v-model="jsonData.caption"
          type="text"
          placeholder="Enter caption"
        />

    <!-- Blocked -->
    <h2>Blocked Options</h2>

    <label for="delete">DELETE:</label>
    <input
      id="delete"
      @input="updateBlockedArray('DELETE', $event.target.value)"
      type="text"
      placeholder="Enter values separated by commas"
    />

    <label for="clone">CLONE:</label>
    <input
      id="clone"
      @input="updateBlockedArray('CLONE', $event.target.value)"
      type="text"
      placeholder="Enter values separated by commas"
    />

    <label for="move">MOVE:</label>
    <input
      id="move"
      @input="updateBlockedArray('MOVE', $event.target.value)"
      type="text"
      placeholder="Enter values separated by commas"
    />

    <label for="block">BLOCK:</label>
    <input
      id="block"
      @input="updateBlockedArray('BLOCK', $event.target.value)"
      type="text"
      placeholder="Enter values separated by commas"
    />

        <section>
          <h2>Groups</h2>
          <div v-for="(group, index) in jsonData.groups" :key="index">
            <label :for="'group-caption-' + index">Group Caption:</label>
            <input
              :id="'group-caption-' + index"
              v-model="group.caption"
              type="text"
              placeholder="Group caption"
            />

            <label :for="'group-comment-' + index">Comment:</label>
            <textarea
              :id="'group-comment-' + index"
              v-model="group.comment"
              placeholder="Group comment"
            ></textarea>
          </div>
        </section>
      </form>

      <h2>Fields</h2>
  <div class="fields_container">
    <div
      v-for="(field, fieldIndex) in jsonData.groups[0].fields"
      :key="fieldIndex"
      class="field_item"
      draggable="true"
      @dragstart="onDragStart(fieldIndex)"
      @dragover.prevent
      @drop="onDrop(fieldIndex)"
    >
      <p>Field {{ fieldIndex + 1 }}</p>

      <!-- Caption -->
      <div class="field_group">
        <label :for="'field-caption-' + fieldIndex">Caption:</label>
        <input
          :id="'field-caption-' + fieldIndex"
          v-model="field.caption"
          type="text"
        />
      </div>

      <!-- ID -->
      <div class="field_group">
        <label :for="'field-id-' + fieldIndex">ID:</label>
        <input
          :id="'field-id-' + fieldIndex"
          v-model="field.id"
          type="text"
        />
      </div>

      <!-- Type -->
      <div class="field_group">
        <label :for="'field-type-' + fieldIndex">Type:</label>
        <select
          :id="'field-type-' + fieldIndex"
          v-model="field.type"
        >
          <option value="input">Input</option>
          <option value="textarea">Textarea</option>
          <option value="background">Background</option>
          <option value="datepicker">Datepicker</option>
          <option value="editor">Editor</option>
          <option value="media">Media</option>
        </select>
      </div>

      <!-- Width -->
      <div class="field_group">
        <label :for="'field-width-' + fieldIndex">Width:</label>
        <input
          :id="'field-width-' + fieldIndex"
          v-model.number="field.width"
          type="number"
          min="1"
          max="12"
        />
      </div>

      <!-- Delete Button -->
      <span @click.prevent="deleteField(fieldIndex)" class="delete_button">
        X
      </span>
    </div>
  </div>
        <BottonAdd @click.prevent="addField">Add Field</BottonAdd>
    </aside>

  </div>
<Footer/>
</template>
<style lang="scss" src="./scss/main.scss"></style>

