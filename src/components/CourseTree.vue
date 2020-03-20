<template>
    <v-card>
    <v-toolbar
      color="primary"
      dark
      flat
    >
      <v-icon>mdi-school</v-icon>
      <v-toolbar-title class="pa-4"> Course Tree for {{ this.currentCourse  }}</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-autocomplete
        flat
        solo-inverted
        hide-details
        prepend-inner-icon="mdi-magnify"
        label="Search"
        class="hidden-sm-and-down"
        v-model="model"
        :items="searchitems"
        :loading="isLoading"
        :search-input.sync="search"
        item-text="query"
        item-value="code"
        placeholder="Start typing a course code to search"
        return-object
      ></v-autocomplete>
    </v-toolbar>
    <v-row
        class="pa-4"
        justify="space-between"
    >
      <v-col cols="2">
        <v-card-text>
          <v-progress-circular
            v-if="loading"
            indeterminate
            color="primary"
    ></v-progress-circular>
          <v-treeview
            v-else
            v-model="selection"
            :items="tree"
            activatable
            :active.sync="active"
            :rounded="true"
            transition
            item-key="name"
            ></v-treeview>
        </v-card-text>
      </v-col>

      <v-divider vertical></v-divider>

      <v-col
        class="d-flex text-center"
      >
          <v-card
            :key="1"
            class="pt-6 mx-auto"
            flat
            max-width="900"
            v-if="!selected && !selected_data && !loading"
          >
            <div
              class="title grey--text text--lighten-1 font-weight-light"
            >
              Select a Course for more info
            </div>
          </v-card>
          <v-card
            :key="1"
            class="pt-6 mx-auto"
            flat
            max-width="900"
            v-else-if="selected && loadingCourse"
          >
            <div
              class="title grey--text text--lighten-1 font-weight-light"
            >
              <v-progress-circular
                indeterminate
                color="primary"
              ></v-progress-circular>
              Loading
            </div>
          </v-card>
          <v-card
            v-else
            :key="1"
            class="pt-6 mx-auto"
            flat
            max-width="900"
          >
            <v-card-text v-if="selected_data">
              <h3 class="headline mb-2">
                <a :href="`/view?course=${selected_data.code}`" target="_blank">{{ selected_data.code }} - {{ selected_data.name }}</a>
                
              </h3>
            </v-card-text>
            <v-divider></v-divider>
            <v-row 
              v-if="selected_data"
              class="text-left"
              tag="v-card-text"
            >
              <v-col class="text-right mr-4 mb-4" tag="strong" cols="2">Description:</v-col>
              <v-col>{{ selected_data.description }}</v-col>
              <!-- <v-col class="text-right mr-2 mb-4" tag="strong" cols="3">description:</v-col>
              <v-col>{{ selected_data.description }}</v-col> -->
              <!-- <v-col class="text-right mr-4 mb-2" tag="strong" cols="5">Website:</v-col> -->
              <!-- <v-col>
                <a :href="`//${selected.website}`" target="_blank">{{ selected.website }}</a>
              </v-col> -->
            </v-row>
            <v-row class="text-left" tag="v-card-text" v-if="selected_data">
              <v-col class="text-right mr-4 mb-2" tag="strong" cols="2">Campus:</v-col>
              <v-col>{{ selected_data.campus }}</v-col> 
            </v-row>
          </v-card>
      </v-col>
    </v-row>
  </v-card>
</template>

<script>
import axios from 'axios';
export default {
    data: () => ({
      currentCourse: null,
      active: [],
      selected_data: null,
      items: [],
      loading: false,
      loadingCourse: false,
      tree : [],
      open: [],
      selection: [],
      descriptionLimit: 60,
      entries: [],
      isLoading: false,
      model: null,
      search: null,
    }),
    computed: {
      selected () {
        if (!this.active.length) return undefined

        const id = this.active[0]
        
        return id
      },
      fields () {
        if (!this.model) return []
        return Object.keys(this.model).map(key => {
          return {
            key,
            value: this.model[key] || 'n/a',
          }
        })
        
      },
      searchitems () {
        return this.entries.map(entry => {
          const query = entry.name.length > this.descriptionLimit
            ? entry.code + " : " + entry.name.slice(0, this.descriptionLimit) + '...'
            : entry.code + " : " + entry.name
          return Object.assign({}, entry, { query })
        })
      },
 
    },
    watch:{
        selected(newValue) {
          (newValue)? this.onUpdate(newValue) : null
          
        },
        '$route': 'fetchData',
        search (val) {
        // Items have already been loaded
        if (this.model && val.length > 10) return
        if (val.length < 4) return 
        // Items have already been requested
        if (this.isLoading) return
        this.isLoading = true
        // Lazily load input items
        axios.get(`http://192.168.1.78:8080/api/search_queries?query=`+val)
          .then(response => {
            // console.log(response.data)
            this.entries = response.data.result
            
          })
          .catch(err => {
            console.log(err)
          })
          .finally(() => (this.isLoading = false))
      },
      model: function(val) {
        console.log(val.code)
        this.fetchData(val.code)
      }
    },
    mounted: function() {
      if (this.$route.query.course) {
        this.course = this.$route.query.course;
      }
  },
  created () {
    this.fetchData(this.$route.query.course)
  },
  methods: {
    onUpdate(selection) {
      this.loadingCourse= true
      axios.get(`http://192.168.1.78:8080/api/search?course=`+selection)
      .then(response => {
        this.loadingCourse = false
        this.selected_data = response.data
      })
      .catch(e => {
        this.errors = e.toString();
      })

    },
    fetchData (val) {
        this.loading = true
        this.currentCourse = null
        axios.get(`http://192.168.1.78:8080/api/tree?course=`+val)
        .then(response => {
          this.loading = false
          this.currentCourse = val
          this.search = null
          this.model = null
          this.onUpdate(val)
          this.tree = [response.data]
        })
        .catch(e => {
          this.errors = e.toString();
        })
    }
  }
  }
</script>
<style scoped>
.v-progress-circular {
  margin: 1rem;
}
</style>
