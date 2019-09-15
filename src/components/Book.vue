<template>
  <div class="container mx-auto py-10">
    <div class="py-2">
    <b-button v-b-toggle.collapse-1 variant="primary">Add A Book</b-button>
    </div>
    <b-collapse id="collapse-1" class="mt-2">
      <b-card w-50>
        <p class="card-text">Enter book details</p>
        <form action="" @submit.prevent="addBook">
          <div class="bg-gray-200 p-4">
            <span class="block text-gray-700 text-center bg-gray-400 px-10 py-2">
              <b-form-input class="input"
              autofocus autocomplete="off"
              placeholder="ISBN"
              v-model="newBook.isbn" ></b-form-input>
            </span>
            <span class="block text-gray-700 text-center bg-gray-400 px-10 py-2 mt-2">
              <b-form-input class="input"
              autofocus autocomplete="off"
              placeholder="Title"
              v-model="newBook.title" ></b-form-input>
            </span>
            <span class="block text-gray-700 text-center bg-gray-400 px-10 py-2 mt-2">
              <b-form-input type="date"
              v-model="newBook.date_published" ></b-form-input>
            </span>
            <span class="block text-gray-700 text-center bg-gray-400 px-10 py-2 mt-2">
              <b-button v-b-toggle.collapse-1>Cancel</b-button>
              <b-button type="submit" variant="primary" size="md">Save book record</b-button>
            </span>
          </div>
        </form>
      </b-card>
    </b-collapse>
    <b-table :items="books" :fields="fields" striped responsive="sm">
      <template v-slot:cell(show_details)="row">
        <b-button size="sm" variant="primary" v-b-modal.modal-prevent-closing @click.prevent="editBook(row.item)">Edit</b-button>
        <b-modal
          v-if="row.item == editedBook"
          id="modal-prevent-closing"
          ref="modal"
          title="Update Book"
          size="md"
          hide-footer
        >
          <form action="" @submit.prevent="updateBook(row.item)">
            <div class="bg-gray-200 p-4">
              <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                ISBN: <b-form-input class="input" v-model="row.item.isbn"></b-form-input>
              </span>
              <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2 mt-2">
                Title: <b-form-input class="input" v-model="row.item.title"></b-form-input>
              </span>
              <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2 mt-2">
                Date Published: <b-form-input type="date" class="input" v-model="row.item.date_published"></b-form-input>
              </span>
              <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2 mt-2">
                <b-button type="submit" variant="primary">Update</b-button>
              </span>
            </div>
          </form>
        </b-modal>
        <b-button size="sm" variant="danger"
          @click.prevent="removeBook(row.item)" class="mr-2">Delete</b-button>
      </template>
    </b-table>
  </div>
</template>

<script>

export default {
  name: 'Book',
  data () {
    return {
      fields: [
        {
          key: 'isbn',
          label: 'ISBN',
          sortable: true
        },
        {
          key: 'title',
          label: 'Title',
          sortable: true
        },
        {
          key: 'authors',
          label: 'Authors'
        },
        {
          key: 'genre',
          label: 'Genre'
        },
        {
          key: 'date_published',
          label: 'Date Published',
          sortable: true
        },
        {
          key: 'show_details',
          label: 'Actions'
        }
      ],
      perPage: 10,
      currentPage: 1,
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
