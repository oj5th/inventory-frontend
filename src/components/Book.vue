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
              placeholder="000-0-000000-00-0"
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
    <b-container fluid>
      <!-- User Interface controls -->
      <b-row>
        <b-col lg="6" class="my-1">
          <b-form-group
            label="Sort"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            label-for="sortBySelect"
            class="mb-0"
          >
            <b-input-group size="sm">
              <b-form-select v-model="sortBy" id="sortBySelect" :options="sortOptions" class="w-75">
                <template v-slot:first>
                  <option value="">-- none --</option>
                </template>
              </b-form-select>
              <b-form-select v-model="sortDesc" size="sm" :disabled="!sortBy" class="w-25">
                <option :value="false">Asc</option>
                <option :value="true">Desc</option>
              </b-form-select>
            </b-input-group>
          </b-form-group>
        </b-col>

        <b-col lg="6" class="my-1">
          <b-form-group
            label="Filter"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            label-for="filterInput"
            class="mb-0"
          >
            <b-input-group size="sm">
              <b-form-input
                v-model="filter"
                type="search"
                id="filterInput"
                placeholder="Type to Search"
              ></b-form-input>
              <b-input-group-append>
                <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
              </b-input-group-append>
            </b-input-group>
          </b-form-group>
        </b-col>

        <b-col sm="5" md="6" class="my-1">
          <b-form-group
            label="Per page"
            label-cols-sm="6"
            label-cols-md="4"
            label-cols-lg="3"
            label-align-sm="right"
            label-size="sm"
            label-for="perPageSelect"
            class="mb-0"
          >
            <b-form-select
              v-model="perPage"
              id="perPageSelect"
              size="sm"
              :options="pageOptions"
            ></b-form-select>
          </b-form-group>
        </b-col>

        <b-col sm="7" md="6" class="my-1">
          <b-pagination
            v-model="currentPage"
            :total-rows="totalRows"
            :per-page="perPage"
            align="fill"
            size="sm"
            class="my-0"
          ></b-pagination>
        </b-col>
      </b-row>

      <!-- Main table element -->
      <b-table
        show-emptyitems
        small
        stacked="md"
        :items="books"
        :fields="fields"
        :current-page="currentPage"
        :per-page="perPage"
        :filter="filter"
        :sort-by.sync="sortBy"
        :sort-desc.sync="sortDesc"
        :sort-direction="sortDirection"
        @filtered="onFiltered"
      >
        <template v-slot:cell(name)="row">
          {{ row.value.first }} {{ row.value.last }}
        </template>

        <template v-slot:cell(show_details)="row">
          <b-button size="sm" variant="primary" v-b-modal.add-author @click.prevent="editBook(row.item)">Add Author</b-button>
          <b-modal
            v-if="row.item == editedBook"
            id="add-author"
            ref="modal"
            title="Add Author"
            size="md"
            hide-footer
          >
            <form action="" @submit.prevent="updateAuthor(row.item)">
              <div class="bg-gray-200 p-4">
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                  First Name: <b-form-input class="input" v-model="row.item.firstname"></b-form-input>
                </span>
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                  Middle Name: <b-form-input class="input" v-model="row.item.middlename"></b-form-input>
                </span>
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                  Last Name: <b-form-input class="input" v-model="row.item.lastname"></b-form-input>
                </span>
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2 mt-2">
                  <b-button type="submit" variant="primary">Save</b-button>
                </span>
              </div>
            </form>
          </b-modal>
          <b-button size="sm" variant="success" v-b-modal.add-category @click.prevent="editBook(row.item)">Add Category</b-button>
          <b-modal
            v-if="row.item == editedBook"
            id="add-category"
            ref="modal"
            title="Add Category"
            size="md"
            hide-footer
          >
            <form action="" @submit.prevent="updateCategory(row.item)">
              <div class="bg-gray-200 p-4">
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                  <select class="select" v-model="row.item.genre_id">
                    <option disabled value="">Select genre</option>
                    <option :value="r.id" v-for="r in row.item.all_genre" :key="r.id">{{ r.name }}</option>
                  </select>
                </span>
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2 mt-2">
                  <b-button type="submit" variant="primary">Save</b-button>
                </span>
              </div>
            </form>
          </b-modal>
          <b-button size="sm" variant="primary" v-b-modal.update-book @click.prevent="editBook(row.item)">Edit</b-button>
          <b-modal
            v-if="row.item == editedBook"
            id="update-book"
            ref="modal"
            title="Update Book"
            size="md"
            hide-footer
          >
            <form action="" @submit.prevent="updateBook(row.item)">
              <div class="bg-gray-200 p-4">
                <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                  ISBN: <b-form-input class="input" placeholder="000-0-000000-00-0" v-model="row.item.isbn"></b-form-input>
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
    </b-container>
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
      editedBook: '',
      totalRows: 1,
      currentPage: 1,
      perPage: 5,
      pageOptions: [5, 10, 15],
      sortBy: '',
      sortDesc: false,
      sortDirection: 'asc',
      filter: null,
      fields: [
        {
          key: 'isbn',
          label: 'ISBN',
          sortable: true,
          sortDirection: 'asc'
        },
        {
          key: 'title',
          label: 'Title',
          sortable: true,
          class: 'text-center'
        },
        {
          key: 'authors',
          label: 'Authors'
        },
        {
          key: 'genres',
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
      ]
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
  computed: {
    sortOptions () {
      // Create an options list from our fields
      return this.fields
        .filter(f => f.sortable)
        .map(f => {
          return { text: f.label, value: f.key }
        })
    }
  },
  mounted () {
    // Set the initial number of items
    this.totalRows = this.books.length
  },
  methods: {
    onFiltered (filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
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
        .catch(error => this.setError(error, 'Please check all required field.'))
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
    },
    updateAuthor (book) {
      this.editedBook = ''
      this.$http.secured.patch(`/api/v1/books/${book.id}`, { book: { authors_attributes: [{firstname: book.firstname, middlename: book.middlename, lastname: book.lastname}] } })
        .catch(error => this.setError(error, 'Cannot update book'))
    },
    updateCategory (book) {
      this.editedBook = ''
      this.$http.secured.patch(`/api/v1/books/${book.id}`, { book: { book_genres_attributes: [{genre_id: book.genre_id}]  } })
        .catch(error => this.setError(error, 'Cannot update book'))
    }
  }
}
</script>
