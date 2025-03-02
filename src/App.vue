<script setup>
import { ref } from 'vue'

const storageName = 'soundslikeatally'

const tallyList = localStorage.getItem(storageName)
  ? JSON.parse(localStorage.getItem(storageName))
  : ref({})
let inputTally = ''

function incrementTally(name) {
  if (tallyList.value[name] >= 2147483648) {
    tallyList.value[name] = 2147483648
    return
  }

  tallyList.value[name]++
}

function decrementTally(name) {
  if (tallyList.value[name] <= 0) {
    return
  }

  tallyList.value[name]--
}

function addNewTallyTrack(name) {
  if (name === '' || Object.keys(tallyList.value).includes(name)) {
    return
  }

  tallyList.value[name] = 0

  updateStorageStorage()

  inputTally = ''
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
      updateStorageStorage()
    })
}

function exportData() {
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

function updateStorageStorage() {
  localStorage.setItem(storageName, JSON.stringify(tallyList.value))
}
</script>

<template>
  <main>
    <h1>That's a Tally</h1>
    <hr />

    <div>
      <h2>Capturing Tallies for:</h2>

      <table>
        <tr v-bind:key="name" v-for="name in Object.keys(tallyList)">
          <td>{{ name }}</td>
          <td>{{ tallyList[name] }}</td>
          <td>
            <button @click="incrementTally(name)">+</button>
            <button @click="decrementTally(name)">-</button>
          </td>
        </tr>
      </table>
      <div>
        <input v-model="inputTally" type="text" />
        <button @click="addNewTallyTrack(inputTally)">+</button>
      </div>
    </div>

    <div
      id="drag"
      @dragover.prevent.stop="dragging = true"
      @dragleave.prevent.stop="dragging = false"
      @drop.prevent="readDragData"
    >
      <p>Drag and drop a previous tally list here or click to upload.</p>
      <input @change="readData" type="file" accept=".json" />
    </div>
    <div>
      <button @click="exportData()">Export JSON</button>
    </div>
  </main>
</template>

<style scoped>
#drag {
  border: 2px solid red;
  height: 100px;
}

@media (min-width: 1024px) {
}
</style>
