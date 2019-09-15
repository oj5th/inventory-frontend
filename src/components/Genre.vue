<template>
  <div class="container mx-auto py-10">
    <div class="py-2">
    <b-button v-b-toggle.collapse-1 variant="primary">Add Book Genre</b-button>
    </div>
    <b-collapse id="collapse-1" class="mt-2">
      <b-card w-50>
        <p class="card-text">Enter genre</p>
        <form action="" @submit.prevent="addGenre">
          <div class="bg-gray-200 p-4">
            <span class="block text-gray-700 text-center bg-gray-400 px-10 py-2">
              <b-form-input class="input"
              autofocus autocomplete="off"
              placeholder="Genre"
              v-model="newGenre.name" ></b-form-input>
            </span>
            <span class="block text-gray-700 text-center bg-gray-400 px-10 py-2 mt-2">
              <b-button v-b-toggle.collapse-1>Cancel</b-button>
              <b-button type="submit" variant="primary" size="md">Save Genre</b-button>
            </span>
          </div>
        </form>
      </b-card>
    </b-collapse>
    <b-table :items="genres" :fields="fields" striped responsive="sm">
      <template v-slot:cell(show_details)="row">
        <b-button size="sm" variant="primary" v-b-modal.update-genre @click.prevent="editGenre(row.item)">Edit</b-button>
        <b-modal
          v-if="row.item == editedGenre"
          id="update-genre"
          ref="modal"
          title="Update Genre"
          size="md"
          hide-footer
        >
          <form action="" @submit.prevent="updateGenre(row.item)">
            <div class="bg-gray-200 p-4">
              <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2">
                Genre: <b-form-input class="input" v-model="row.item.name"></b-form-input>
              </span>
              <span class="block text-gray-700 text-center bg-gray-400 px-4 py-2 mt-2">
                <b-button type="submit" variant="primary">Update</b-button>
              </span>
            </div>
          </form>
        </b-modal>
        <b-button size="sm" variant="danger"
          @click.prevent="removeGenre(row.item)" class="mr-2">Delete</b-button>
      </template>
    </b-table>
  </div>
</template>

<script>

export default {
  name: 'Genre',
  data () {
    return {
      fields: [
        {
          key: 'name',
          label: 'Genre',
          sortable: true
        },
        {
          key: 'show_details',
          label: 'Actions'
        }
      ],
      perPage: 10,
      currentPage: 1,
      genres: [],
      newGenre: [],
      error: '',
      editedGenre: ''
    }
  },
  created () {
    if (!localStorage.signedIn) {
      this.$router.replace('/')
    } else {
      this.$http.secured.get('/api/v1/genres')
        .then(response => { this.genres = response.data })
        .catch(error => this.setError(error, 'Something went wrong'))
    }
  },
  methods: {
    setError (error, text) {
      this.error = (error.response && error.response.data && error.response.data.error) || text
    },
    addGenre () {
      const value = this.newGenre
      if (!value) {
        return
      }
      this.$http.secured.post('/api/v1/genres/', { genre: { name: this.newGenre.name } })

        .then(response => {
          this.genres.push(response.data)
          this.newGenre = ''
        })
        .catch(error => this.setError(error, 'Cannot add book'))
    },
    removeGenre (genre) {
      this.$http.secured.delete(`/api/v1/genres/${genre.id}`)
        .then(response => {
          this.genres.splice(this.genres.indexOf(genre), 1)
        })
        .catch(error => this.setError(error, 'Cannot remove book'))
    },
    editGenre (genre) {
      this.editedGenre = genre
    },
    updateGenre (genre) {
      this.editedGenre = ''
      this.$http.secured.patch(`/api/v1/genres/${genre.id}`, { genre: { name: genre.name } })
        .catch(error => this.setError(error, 'Cannot update book'))
    }
  }
}
</script>
