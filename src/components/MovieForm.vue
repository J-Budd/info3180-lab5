<template>
  <form id="movieForm" @submit.prevent="saveMovie" enctype="multipart/form-data">
    <div v-if="message" class="alert" :class="{'alert-success': success, 'alert-danger': !success}">
      {{ message }}
    </div>
    
    <div class="form-group mb-3">
      <label for="title" class="form-label">Movie Title</label>
      <input v-model="title" type="text" name="title" class="form-control" />
    </div>

    <div class="form-group mb-3">
      <label for="description" class="form-label">Description</label>
      <textarea v-model="description" name="description" class="form-control"></textarea>
    </div>

    <div class="form-group mb-3">
      <label for="poster" class="form-label">Poster</label>
      <input ref="poster" type="file" name="poster" class="form-control" />
    </div>

    <button type="submit" class="btn btn-primary">Add Movie</button>

    <ul v-if="errors.length" class="text-danger mt-2">
      <li v-for="(err, index) in errors" :key="index">{{ err }}</li>
    </ul>
  </form>
</template>

<script setup>
import { ref, onMounted } from 'vue'

let title = ref("")
let description = ref("")
let poster = ref(null)

let message = ref("")
let success = ref(false)
let errors = ref([])
let csrf_token = ref("")

function getCsrfToken() {
  fetch('/api/v1/csrf-token')
    .then(res => res.json())
    .then(data => {
      csrf_token.value = data.csrf_token
    })
}

onMounted(() => {
  getCsrfToken()
})

function saveMovie() {
  const form = new FormData()
  form.append('title', title.value)
  form.append('description', description.value)
  form.append('poster', poster.value.files[0])

  fetch('/api/v1/movies', {
    method: 'POST',
    headers: {
      'X-CSRFToken': csrf_token.value
    },
    body: form
  })
  .then(res => res.json())
  .then(data => {
    if (data.message) {
      message.value = data.message
      success.value = true
      errors.value = []
      // clear form
      title.value = ""
      description.value = ""
      poster.value.value = ""
    } else if (data.errors) {
      message.value = ""
      success.value = false
      errors.value = data.errors
    }
  })
  .catch(err => {
    message.value = "An error occurred"
    success.value = false
    errors.value = []
    console.error(err)
  })
}
</script>

<style scoped>
.alert {
  padding: 1em;
  margin-bottom: 1em;
  border-radius: 4px;
}
.alert-success {
  background-color: #d1e7dd;
  color: #0f5132;
}
.alert-danger {
  background-color: #f8d7da;
  color: #842029;
}
</style>
