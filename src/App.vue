<script setup>
import { ref } from 'vue';
import JSZip from 'jszip';

// Declarar shortcodeName
const shortcodeName = ref('');

// Estado inicial del JSON
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

// Lista de shortcodes
const shortcodes = ref([]);

// Función para agregar nuevos campos
const addField = () => {
  jsonData.value.groups[0].fields.push({
    caption: '',
    id: '',
    type: 'input',
    blocked: [],
    width: 6,
  });
};

// Función para eliminar un campo por su índice
const deleteField = (index) => {
  jsonData.value.groups[0].fields.splice(index, 1);
};

// Estado para arrastrar y soltar
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
  // Limpiar los valores, separarlos por coma y asegurarse de que sean números
  const newValues = value
    .split(',')
    .map((item) => parseFloat(item.trim()))
    .filter((item) => !isNaN(item)); // Filtra valores no numéricos

  // Sobrescribe el array con los nuevos valores
  jsonData.value.blocked[key] = newValues;
};


const addShortcode = () => {
  if (!shortcodeName.value.trim()) {
    alert('Please provide a shortcode name.');
    return;
  }

  // Agregar el shortcode a la lista
  shortcodes.value.push({
    name: shortcodeName.value,
    data: jsonData.value,
  });

  // Reiniciar el formulario
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

  // Crear un archivo ZIP para descargar todos los shortcodes
  const zip = new JSZip();

  shortcodes.value.forEach((shortcode) => {
    const jsonBlob = new Blob([formatJson(shortcode.data)], {
      type: 'application/json',
    });
    zip.file(`${shortcode.name}.json`, jsonBlob);
  });

  zip.generateAsync({ type: 'blob' }).then((content) => {
    const link = document.createElement('a');
    link.href = URL.createObjectURL(content);
    link.download = 'shortcodes.zip';
    link.click();
  });
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
    <!-- Previsualización del JSON -->
    <main>
      <h2>JSON Preview</h2>
      <pre>{{ formatJson(jsonData) }}</pre>
    </main>
    <button class="button-create_shortcode button_add_list" @click.prevent="addShortcode">
      <span class="button__text">Add Shortcode</span>
      <span class="button__icon"><svg xmlns="http://www.w3.org/2000/svg" width="24" viewBox="0 0 24 24" stroke-width="2" stroke-linejoin="round" stroke-linecap="round" stroke="currentColor" height="24" fill="none" class="svg"><line y2="19" y1="5" x2="12" x1="12">
      </line><line y2="12" y1="12" x2="19" x1="5"></line></svg></span>
    </button>
    <!-- Formulario para crear/editar JSON -->
    <aside class="aside_tools">
      <!-- Lista de shortcodes -->
      <div class="aside_shortcodes">
        <h2>Shortcodes List</h2>
        <ul>
          <li v-for="(shortcode, index) in shortcodes" :key="index">
            {{ shortcode.name }}
          </li>
        </ul>
        <button class="button-create_shortcode" @click.prevent="downloadJsonFiles">
          <span class="button__text">Download All Shortcodes</span>
          <span class="button__icon"><svg xmlns="http://www.w3.org/2000/svg" width="24" viewBox="0 0 24 24" stroke-width="2" stroke-linejoin="round" stroke-linecap="round" stroke="currentColor" height="24" fill="none" class="svg"><line y2="19" y1="5" x2="12" x1="12">
          </line><line y2="12" y1="12" x2="19" x1="5"></line></svg></span>
        </button>
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

    <!-- Bloqueado -->
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

        <button class="button-create_shortcode" @click.prevent="addField">
        <span class="button__text">Crear</span>
        <span class="button__icon"><svg xmlns="http://www.w3.org/2000/svg" width="24" viewBox="0 0 24 24" stroke-width="2" stroke-linejoin="round" stroke-linecap="round" stroke="currentColor" height="24" fill="none" class="svg"><line y2="19" y1="5" x2="12" x1="12">
        </line><line y2="12" y1="12" x2="19" x1="5"></line></svg></span>
      </button>
    </aside>

  </div>

</template>

