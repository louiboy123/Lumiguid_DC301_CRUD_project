<script setup>
import { computed, ref } from 'vue'

const storageKey = 'book-tracker-books'

const defaultForm = () => ({
  title: '',
  author: '',
  genre: 'Fantasy',
  status: 'Want to Read',
  notes: ''
})

const loadBooks = () => {
  const savedBooks = localStorage.getItem(storageKey)

  if (!savedBooks) {
    return [
      {
        id: 1,
        title: 'Atomic Habits',
        author: 'James Clear',
        genre: 'Self-Improvement',
        status: 'Reading',
        notes: 'A practical guide to building better routines.'
      },
      {
        id: 2,
        title: 'The Hobbit',
        author: 'J.R.R. Tolkien',
        genre: 'Fantasy',
        status: 'Read',
        notes: 'A timeless adventure with a warm, imaginative tone.'
      },
      {
        id: 3,
        title: 'Clean Code',
        author: 'Robert C. Martin',
        genre: 'Technology',
        status: 'Want to Read',
        notes: 'Great resource for writing maintainable software.'
      }
    ]
  }

  try {
    const parsed = JSON.parse(savedBooks)
    return Array.isArray(parsed) && parsed.length ? parsed : []
  } catch (error) {
    return []
  }
}

const books = ref(loadBooks())
const form = ref(defaultForm())
const editingId = ref(null)
const filter = ref('All')

const filterOptions = ['All', 'Read', 'Reading', 'Want to Read']

const filteredBooks = computed(() => {
  if (filter.value === 'All') {
    return books.value
  }

  return books.value.filter((book) => book.status === filter.value)
})

const counts = computed(() => ({
  all: books.value.length,
  read: books.value.filter((book) => book.status === 'Read').length,
  reading: books.value.filter((book) => book.status === 'Reading').length,
  want: books.value.filter((book) => book.status === 'Want to Read').length
}))

const isEditing = computed(() => editingId.value !== null)

function saveBooks() {
  localStorage.setItem(storageKey, JSON.stringify(books.value))
}

function resetForm() {
  form.value = defaultForm()
  editingId.value = null
}

function submitBook() {
  const title = form.value.title.trim()
  const author = form.value.author.trim()

  if (!title || !author) {
    return
  }

  const bookData = {
    ...form.value,
    title,
    author,
    notes: form.value.notes.trim()
  }

  if (editingId.value !== null) {
    books.value = books.value.map((book) =>
      book.id === editingId.value ? { ...book, ...bookData } : book
    )
  } else {
    books.value.unshift({
      id: Date.now(),
      ...bookData
    })
  }

  saveBooks()
  resetForm()
}

function editBook(book) {
  editingId.value = book.id
  form.value = { ...book }
}

function deleteBook(id) {
  books.value = books.value.filter((book) => book.id !== id)

  if (editingId.value === id) {
    resetForm()
  }

  saveBooks()
}

function clearAllBooks() {
  if (!books.value.length) {
    return
  }

  const confirmed = window.confirm('Delete all books from your shelf?')

  if (confirmed) {
    books.value = []
    saveBooks()
    resetForm()
  }
}
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div>
        <p class="eyebrow">Reading dashboard</p>
        <h1>Book Tracker</h1>
      </div>
      <div class="header-badge">
        <span>{{ counts.all }}</span>
        <small>Books</small>
      </div>
    </header>

    <main class="content">
      <section class="panel form-panel">
        <div class="panel-header">
          <h2>{{ isEditing ? 'Update book' : 'Add a new book' }}</h2>
        </div>

        <form class="book-form" @submit.prevent="submitBook">
          <div class="field-row">
            <label>
              <span>Title</span>
              <input v-model="form.title" type="text" placeholder="Enter book title" />
            </label>
            <label>
              <span>Author</span>
              <input v-model="form.author" type="text" placeholder="Enter author name" />
            </label>
          </div>

          <div class="field-row">
            <label>
              <span>Genre</span>
              <select v-model="form.genre">
                <option>Fantasy</option>
                <option>Science Fiction</option>
                <option>Technology</option>
                <option>Self-Improvement</option>
                <option>Biography</option>
                <option>Romance</option>
                <option>History</option>
              </select>
            </label>

            <label>
              <span>Status</span>
              <select v-model="form.status">
                <option>Read</option>
                <option>Reading</option>
                <option>Want to Read</option>
              </select>
            </label>
          </div>

          <label>
            <span>Notes</span>
            <textarea v-model="form.notes" rows="4" placeholder="Write a quick note or reminder..."></textarea>
          </label>

          <div class="form-actions">
            <button type="submit" class="primary-btn">
              {{ isEditing ? 'Save changes' : 'Add book' }}
            </button>
            <button type="button" class="secondary-btn" @click="resetForm">
              Clear form
            </button>
          </div>
        </form>
      </section>

      <section class="panel list-panel">
        <div class="panel-header list-header">
          <h2>My shelf</h2>

          <div class="toolbar">
            <label>
              <span>Filter</span>
              <select v-model="filter">
                <option v-for="option in filterOptions" :key="option" :value="option">
                  {{ option }}
                </option>
              </select>
            </label>
            <button type="button" class="ghost-btn" @click="clearAllBooks">Clear all</button>
          </div>
        </div>

        <div class="stats-grid">
          <div class="stat-card">
            <strong>{{ counts.all }}</strong>
            <span>Total</span>
          </div>
          <div class="stat-card">
            <strong>{{ counts.read }}</strong>
            <span>Read</span>
          </div>
          <div class="stat-card">
            <strong>{{ counts.reading }}</strong>
            <span>Reading</span>
          </div>
          <div class="stat-card">
            <strong>{{ counts.want }}</strong>
            <span>To read</span>
          </div>
        </div>

        <ul class="book-list">
          <li v-for="book in filteredBooks" :key="book.id" class="book-item">
            <div class="book-content">
              <div class="book-meta">
                <span class="genre-tag">{{ book.genre }}</span>
                <span class="status-tag" :class="book.status.toLowerCase().replace(/\s+/g, '-')">
                  {{ book.status }}
                </span>
              </div>
              <h3>{{ book.title }}</h3>
              <p class="author-name">by {{ book.author }}</p>
              <p v-if="book.notes" class="notes">{{ book.notes }}</p>
            </div>

            <div class="item-actions">
              <button type="button" class="secondary-btn" @click="editBook(book)">Edit</button>
              <button type="button" class="danger-btn" @click="deleteBook(book.id)">Delete</button>
            </div>
          </li>

          <li v-if="!filteredBooks.length" class="empty-state">
            No books found for this filter.
          </li>
        </ul>
      </section>
    </main>
  </div>
</template>
