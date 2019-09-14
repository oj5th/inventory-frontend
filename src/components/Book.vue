<template>
  <div class="max-w-md m-auto py-10">
    <div class="text-red" v-if="error">{{ error }}</div>
    <h3 class="font-mono font-regular text-3xl mb-4">Add a book</h3>
    <form action="" @submit.prevent="addBook">
      <div class="mb-6">
        <input class="input"
          autofocus autocomplete="off"
          placeholder="ISBN"
          v-model="newBook.isbn" />
      </div>
      <div class="mb-6">
        <input class="input"
          autofocus autocomplete="off"
          placeholder="Title"
          v-model="newBook.title" />
      </div>
      <div class="mb-6">
        <input class="date"
          v-model="newBook.date_published" />
      </div>
      <input type="submit" value="Save" class="font-sans font-bold px-4 rounded cursor-pointer no-underline bg-green hover:bg-green-dark block w-full py-4 text-white items-center justify-center" />
    </form>

    <hr class="border border-grey-light my-6" />

    <ul class="list-reset mt-4">
      <li class="py-4" v-for="book in books" :key="book.id" :book="book">

        <div class="flex items-center justify-between flex-wrap">
          <p class="block flex-1 font-mono font-semibold flex items-center ">
            {{ book.isbn }}
          </p>

          <p class="block flex-1 font-mono font-semibold flex items-center ">
            {{ book.title }}
          </p>

          <p class="block flex-1 font-mono font-semibold flex items-center ">
            {{ book.date_published }}
          </p>

          <button class="bg-tranparent text-sm hover:bg-blue hover:text-white text-blue border border-blue no-underline font-bold py-2 px-4 mr-2 rounded"
          @click.prevent="editBook(book)">Edit</button>

          <button class="bg-transprent text-sm hover:bg-red text-red hover:text-white no-underline font-bold py-2 px-4 rounded border border-red"
         @click.prevent="removeBook(book)">Delete</button>
        </div>

        <div v-if="book == editedBook">
          <form action="" @submit.prevent="updateBook(book)">
            <div class="mb-6 p-4 bg-white rounded border border-grey-light mt-4">
              <input class="input" v-model="book.isbn" />
              <input class="input" v-model="book.title" />
              <input class="input" v-model="book.date_published" />
              <input type="submit" value="Update" class=" my-2 bg-transparent text-sm hover:bg-blue hover:text-white text-blue border border-blue no-underline font-bold py-2 px-4 rounded cursor-pointer">
            </div>
          </form>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Book',
  data () {
    return {
      books: [],
      newBook: [],
      error: '',
      editedBook: ''
    }
  },
  created () {
    if (!localStorage.signedIn) {
      this.$router.replace('/')
    } else {
      this.$http.secured.get('/api/v1/books')
        .then(response => { this.books = response.data })
        .catch(error => this.setError(error, 'Something went wrong'))
    }
  },
  methods: {
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
    },
    addBook () {
      const value = this.newBook
      if (!value) {
        return
      }
      this.$http.secured.post('/api/v1/books/', { book: { isbn: this.newBook.isbn, title: this.newBook.title, date_published: this.newBook.date_published } })

        .then(response => {
          this.books.push(response.data)
          this.newBook = ''
        })
        .catch(error => this.setError(error, 'Cannot add book'))
    },
    removeBook (book) {
      this.$http.secured.delete(`/api/v1/books/${book.id}`)
        .then(response => {
          this.books.splice(this.books.indexOf(book), 1)
        })
        .catch(error => this.setError(error, 'Cannot remove book'))
    },
    editBook (book) {
      this.editedBook = book
    },
    updateBook (book) {
      this.editedBook = ''
      this.$http.secured.patch(`/api/v1/books/${book.id}`, { book: { isbn: book.isbn, title: book.title, date_published: book.date_published } })
        .catch(error => this.setError(error, 'Cannot update book'))
    }
  }
}
</script>
