<template>
  <div class="container mt-5">
    <h1 class="text-center mb-4">Моя коллекция</h1>

    <div class="row">
      <div class="col-md-8 offset-md-2">
        <form @submit.prevent="saveBook" class="card mb-4 p-4">
          <h5 class="mb-3">{{ editingId ? 'Редактировать карточку' : 'Добавить новую карточку' }}</h5>
          
          <div class="mb-3">
            <input
              v-model="formData.title"
              type="text"
              class="form-control"
              placeholder="Название"
              required
            >
          </div>
          
          <div class="mb-3">
            <textarea
              v-model="formData.description"
              class="form-control"
              rows="2"
              placeholder="Описание"
            ></textarea>
          </div>
          
          <div class="mb-3">
            <input
              v-model="formData.imageUrl"
              type="url"
              class="form-control"
              placeholder="URL фото"
            >
          </div>
          
          <div class="d-flex gap-2">
            <button class="btn btn-primary" type="submit">
              {{ editingId ? 'Сохранить' : 'Добавить' }}
            </button>
            <button 
              type="button" 
              class="btn btn-secondary" 
              @click="resetForm"
            >
              Отмена
            </button>
          </div>
        </form>
      </div>
    </div>

    <div class="row mb-4">
      <div class="col-md-4 offset-md-4">
        <div class="btn-group w-100" role="group">
          <button 
            type="button" 
            class="btn btn-outline-primary" 
            :class="{ active: sortOrder === 'asc' }"
            @click="sortBooks('asc')"
          >
            По алфавиту (А→Я)
          </button>
          <button 
            type="button" 
            class="btn btn-outline-primary" 
            :class="{ active: sortOrder === 'desc' }"
            @click="sortBooks('desc')"
          >
            По алфавиту (Я→А)
          </button>
        </div>
      </div>
    </div>

    <div class="row">
      <div
        v-for="book in sortedBooks"
        :key="book.id"
        class="col-md-4 mb-3"
      >
        <div class="card h-100">
          <img 
            v-if="book.imageUrl" 
            :src="book.imageUrl" 
            class="card-img-top"
            :alt="book.title"
            style="height: 200px; object-fit: cover;"
            @error="handleImageError(book)"
          >
          <div v-else class="card-img-top bg-light d-flex align-items-center justify-content-center" style="height: 200px;">
            <span class="text-muted">Нет фото</span>
          </div>
          
          <div class="card-body">
            <h5 class="card-title">{{ book.title }}</h5>
            <p class="card-text" v-if="book.description">{{ book.description }}</p>
            <p class="card-text text-muted" v-else>Нет описания</p>
          </div>
          
          <div class="card-footer bg-transparent d-flex gap-2">
            <button
              @click="editBook(book)"
              class="btn btn-sm btn-warning"
            >
              Редактировать
            </button>
            <button
              @click="removeBook(book.id)"
              class="btn btn-sm btn-danger"
            >
              Удалить
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <div v-if="sortedBooks.length === 0" class="text-center text-muted mt-5">
      <p>Коллекция пуста. Добавьте первую карточку!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const books = ref([])
const formData = ref({
  title: '',
  description: '',
  imageUrl: ''
})
const editingId = ref(null)
const sortOrder = ref('asc') // 'asc' или 'desc'

const resetForm = () => {
  formData.value = {
    title: '',
    description: '',
    imageUrl: ''
  }
  editingId.value = null
}

const saveBook = () => {
  if (!formData.value.title.trim()) return
  
  if (editingId.value) {
    // Режим редактирования - обновляем существующую карточку
    const index = books.value.findIndex(book => book.id === editingId.value)
    if (index !== -1) {
      books.value[index] = {
        ...books.value[index],
        title: formData.value.title.trim(),
        description: formData.value.description.trim(),
        imageUrl: formData.value.imageUrl.trim()
      }
    }
  } else {
    // Режим добавления - создаём новую карточку
    books.value.push({
      id: Date.now(),
      title: formData.value.title.trim(),
      description: formData.value.description.trim(),
      imageUrl: formData.value.imageUrl.trim()
    })
  }
  
  resetForm()
}

const editBook = (book) => {
  editingId.value = book.id
  formData.value = {
    title: book.title,
    description: book.description || '',
    imageUrl: book.imageUrl || ''
  }
  // Прокручиваем страницу к форме
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const removeBook = (id) => {
  if (confirm('Вы уверены, что хотите удалить эту карточку?')) {
    books.value = books.value.filter(book => book.id !== id)
    // Если удаляем редактируемую карточку, сбрасываем форму
    if (editingId.value === id) {
      resetForm()
    }
  }
}

const sortBooks = (order) => {
  sortOrder.value = order
}

const sortedBooks = computed(() => {
  const sorted = [...books.value]
  
  if (sortOrder.value === 'asc') {
    return sorted.sort((a, b) => a.title.localeCompare(b.title, 'ru'))
  } else if (sortOrder.value === 'desc') {
    return sorted.sort((a, b) => b.title.localeCompare(a.title, 'ru'))
  }
  
  return sorted
})

const handleImageError = (book) => {
  book.imageUrl = ''
}
</script>