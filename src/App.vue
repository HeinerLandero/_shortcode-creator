<script setup>
import { ref } from 'vue';

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
  jsonData.value.blocked[key] = value
    .split(',')
    .map((item) => parseFloat(item.trim()))
    .filter((item) => !isNaN(item)); // Filtra valores no numéricos
};

const downloadJsonFile = () => {
  if (!shortcodeName.value.trim()) {
    alert('Please provide a shortcode name.');
    return;
  }

  // Convertir el JSON a una cadena
  const jsonBlob = new Blob([JSON.stringify(jsonData.value, null, 2)], {
    type: 'application/json',
  });

  // Crear un enlace para la descarga
  const link = document.createElement('a');
  link.href = URL.createObjectURL(jsonBlob);
  link.download = `${shortcodeName.value}.json`; // Nombre del archivo descargado
  link.click(); // Disparar la descarga
};
</script>

<template>
  <h1>Shortcode Creator</h1>
  <div class="wrapper">
    <!-- Previsualización del JSON -->
    <main>
      <h2>JSON Preview</h2>
      <pre>{{ JSON.stringify(jsonData, null, 2) }}</pre>
      <button class="button-create_shortcode" @click.prevent="downloadJsonFile">
        <span class="button__text">Download Shortcode</span>
        <span class="button__icon"><svg xmlns="http://www.w3.org/2000/svg" width="24" viewBox="0 0 24 24" stroke-width="2" stroke-linejoin="round" stroke-linecap="round" stroke="currentColor" height="24" fill="none" class="svg"><line y2="19" y1="5" x2="12" x1="12">
        </line><line y2="12" y1="12" x2="19" x1="5"></line></svg></span>
      </button>
    </main>

    <!-- Formulario para crear/editar JSON -->
    <aside class="aside_tools">
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

      <button class="button-create" @click.prevent="addField">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 12h15m0 0l-6.75-6.75M19.5 12l-6.75 6.75"></path>
        </svg>
        Crear</button>
    </aside>
  </div>

</template>

<style scoped>

</style>
