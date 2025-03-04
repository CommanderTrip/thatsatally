<!--  eslint-disable-next-line vue/block-lang -->
<script setup>
import { ref } from 'vue'

const storageName = 'soundslikeatally'

const tallyList = localStorage.getItem(storageName)
  ? ref(JSON.parse(localStorage.getItem(storageName)))
  : ref({})
let inputTally = ''
const oldEditName = ref('')

function incrementTally(name) {
  if (tallyList.value[name] >= 2147483648) {
    tallyList.value[name] = 2147483648
    return
  }

  tallyList.value[name]++
  updateStorage()
}

function decrementTally(name) {
  if (tallyList.value[name] <= 0) {
    return
  }

  tallyList.value[name]--
  updateStorage()
}

function addNewTallyTrack(name) {
  if (name === '' || Object.keys(tallyList.value).includes(name)) {
    return
  }

  tallyList.value[name] = 0
  inputTally = ''
  updateStorage()
}

function readDragData(event) {
  loadData(URL.createObjectURL(event.dataTransfer.files[0]))
}

function readData(event) {
  loadData(URL.createObjectURL(event.target.files[0]))
}

function loadData(fileUrl) {
  fetch(fileUrl)
    .then((res) => res.json())
    .then((tallies) => {
      tallyList.value = tallies
      updateStorage()
    })
}

function exportData() {
  if (Object.keys(tallyList.value).length === 0) {
    return
  }

  // Create download
  const data = 'data:text/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(tallyList.value))
  const download = document.createElement('a')
  download.setAttribute('href', data)
  download.setAttribute('download', 'thatsatally.json')
  download.style.display = 'none'

  // Add it to the body, initiate download, then remove it from the page
  document.body.appendChild(download)
  download.click()
  document.body.removeChild(download)
}

function updateStorage() {
  localStorage.setItem(storageName, JSON.stringify(tallyList.value))
}

function editVisible(name, becomeVisible) {
  if (becomeVisible) {
    document.getElementById(name + '-remove-icon').style.display = 'block'
  } else {
    document.getElementById(name + '-remove-icon').style.display = 'none'
  }
}

function remove(name) {
  const newList = { ...tallyList.value }
  console.log(newList)
  delete newList[name]
  tallyList.value = newList
  updateStorage()
}
</script>

<template>
  <main>
    <div class="flex-col preamble">
      <em>ever degrade yourself...</em>
      <h1>That's a Tally</h1>
    </div>

    <hr />

    <div>
      <div id="capture-header">
        <h2>Capturing Tallies for:</h2>
        <input
          v-model="inputTally"
          type="text"
          placeholder="Add Name"
          @keypress.enter="addNewTallyTrack(inputTally)"
        />
        <button @click="addNewTallyTrack(inputTally, $event)">+</button>
      </div>

      <table>
        <tr v-bind:key="name" v-for="name in Object.keys(tallyList)">
          <td>
            <div
              class="tally-name"
              @mouseenter="editVisible(name, true)"
              @mouseleave="editVisible(name, false)"
            >
              <img
                :id="name + '-remove-icon'"
                class="edit"
                src="./assets/remove.svg"
                alt="edit"
                @click="remove(name)"
              />
              <p>{{ name }}</p>
            </div>
          </td>
          <td>{{ tallyList[name] }}</td>
          <td>
            <button class="table-button" @click="incrementTally(name)">+</button>
            <button class="table-button" @click="decrementTally(name)">-</button>
          </td>
        </tr>
      </table>
    </div>

    <div id="import-export-group" class="flex-col">
      <div id="drag" class="flex-col" @drop.prevent="readDragData">
        <img class="full-width" src="./assets/upload.svg" alt="Upload" />
        <p>Drag and drop a previous tally list here or click to upload.</p>
        <input @change="readData" type="file" accept=".json" />
      </div>
      <div>
        <button class="table-button" @click="exportData()">Export Tally list</button>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  width: 100%;
}

table {
  width: 100%;
  margin-block: 2rem;
}

tr {
  text-align: end;
}

.preamble > em {
  margin-bottom: -0.5rem;
}

.table-button {
  background-color: transparent;
  color: white;
  padding: 1rem 2rem;
  cursor: pointer;
}

.table-button:hover {
  background-color: grey;
}

.full-width {
  width: 64px;
}

.tally-name {
  display: flex;
  flex-direction: row;
}

.tally-name > .edit {
  margin-right: 1rem;
  width: 24px;
  cursor: pointer;
}

.edit {
  display: none;
}

.hidden {
  display: none;
}

#capture-header {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-top: 0.5rem;
}

#capture-header > input {
  margin-left: 2rem;
  line-height: 1.5rem;
  width: clamp(5rem, 50%, 10rem);
}

#capture-header > button {
  line-height: 1.5rem;
}

#import-export-group {
  align-items: flex-end;
  margin-top: 2rem;
}

#import-export-group button {
  margin-top: 1rem;
}

#drag {
  align-items: center;
  padding: 1rem;
  border: 2px dashed #808080;
  width: 100%;
}

.flex-col {
  display: flex;
  flex-direction: column;
}

@media (fecolormatrix-width: 1024px) {
}
</style>
