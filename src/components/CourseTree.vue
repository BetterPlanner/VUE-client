<template>
  <v-card>
    <v-toolbar color="primary" dark flat>
      <v-icon>mdi-school</v-icon>
      <v-toolbar-title class="pa-4">
        Course Tree for {{ this.currentCourse }}</v-toolbar-title
      >
      <v-spacer></v-spacer>
      <v-autocomplete
        flat
        solo-inverted
        hide-details
        prepend-inner-icon="mdi-magnify"
        label="Search"
        class="hidden-sm-and-down"
        v-model="model"
        :items="getCourse"
        :loading="$apollo.loading"
        :search-input.sync="search"
        item-text="name"
        no-filter
        item-value="code"
        placeholder="Start typing a course code to search"
        return-object
      >
        <template v-slot:no-data>
          <v-list-item>
            <v-list-item-title>
              Search for a course by code or name
            </v-list-item-title>
          </v-list-item>
        </template>
        <template v-slot:item="{ item }">
          <v-list-item-avatar
            color="indigo"
            class="headline font-weight-light white--text"
          >
            {{ item.code.charAt(0) + item.code.charAt(1) }}
          </v-list-item-avatar>
          <v-list-item-content v-if="item">
            <v-list-item-title v-text="item.name"></v-list-item-title>
            <v-list-item-subtitle v-text="item.code"></v-list-item-subtitle>
          </v-list-item-content>
        </template>
      </v-autocomplete>
    </v-toolbar>
    <v-row class="pa-4" justify="space-between">
      <v-col cols="2">
        <v-card-text>
          <v-progress-circular
            v-if="loading"
            indeterminate
            color="primary"
          ></v-progress-circular>
          <v-treeview
            v-else
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
      <v-col class="d-flex text-center">
        <v-card
          :key="1"
          class="pt-6 mx-auto"
          flat
          max-width="900"
          v-if="selected && $apollo.loading"
        >
          <div class="title grey--text text--lighten-1 font-weight-light">
            <v-progress-circular
              indeterminate
              color="primary"
            ></v-progress-circular>
            Loading
          </div>
        </v-card>
        <v-card v-else :key="1" class="pt-6 mx-auto" flat max-width="900">
          <v-card-text v-if="getData && getData.length > 0">
            <h3 class="headline mb-2">
              <a :href="`/view?course=${getData[0].code}`" target="_blank"
                >{{ getData[0].code }} - {{ getData[0].name }}</a
              >
            </h3>
          </v-card-text>
          <v-divider></v-divider>
          <v-row
            v-if="getData && getData.length > 0"
            class="text-left"
            tag="v-card-text"
          >
            <v-col class="text-left mr-1 mb-2" tag="strong" cols="2"
              >Description:</v-col
            >
            <v-col>{{ getData[0].description }}</v-col>
            <!-- <v-col class="text-right mr-4 mb-2" tag="strong" cols="5">Website:</v-col> -->
            <!-- <v-col>
                <a :href="`//${selected.website}`" target="_blank">{{ selected.website }}</a>
              </v-col> -->
          </v-row>
          <v-row
            class="text-left"
            tag="v-card-text"
            v-if="getData && getData.length > 0"
          >
            <v-col class="text-left mr-1 mb-2" tag="strong" cols="2"
              >Campus:</v-col
            >
            <v-col>{{ getData[0].campus }}</v-col>
          </v-row>
          <v-row
            class="text-left"
            tag="v-card-text"
            v-if="
              getData && getData.length > 0 && getData[0].exclusions.length > 0
            "
          >
            <v-col
              class="text-left mr-1 mb-2"
              tag="strong"
              cols="2"
              v-if="
                getData &&
                  getData.length > 0 &&
                  getData[0].exclusions.length > 0
              "
              >Exclusions:</v-col
            >
            <v-col>{{ getData[0].exclusions }}</v-col>
          </v-row>
          <v-row
            class="text-left"
            tag="v-card-text"
            v-if="
              getData &&
                getData.length > 0 &&
                getData[0].prerequisites.length > 0
            "
          >
            <v-col class="text-left mr-1 mb-2" tag="strong" cols="2"
              >Prerequisites:</v-col
            >
            <v-col>{{ getData[0].prerequisites }}</v-col>
          </v-row>
          <v-row
            class="text-left"
            tag="v-card-text"
            v-if="getData && getData.length > 0 && getData[0].required_for"
          >
            <v-col
              class="text-left mr-1 mb-2"
              tag="strong"
              cols="2"
              v-if="
                getData &&
                  getData.length > 0 &&
                  getData[0].required_for.length > 0
              "
              >Necessary for:</v-col
            >
            <v-col
              ><div v-for="item of getData[0].required_for" v-bind:key="item">
                <a :href="`/tree?course=${item}`">{{ item }}</a>
              </div></v-col
            >
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </v-card>
</template>

<script>
import axios from "axios";
import gql from "graphql-tag";
export default {
  apollo: {
    getCourse: {
      query: gql`
        query getCourse($search: String!) {
          getCourse(search: $search) {
            code
            name
          }
        }
      `,
      variables() {
        return {
          search: this.search
        };
      },
      skip() {
        return !this.search ? true : this.search.length < 3 ? true : false;
      }
    },
    getData: {
      query: gql`
        query getCourse($search: String!) {
          getData: getCourse(search: $search) {
            code
            name
            description
            campus
            prerequisites
            prerequisites_array
            required_for
            exclusions
          }
        }
      `,
      variables() {
        return {
          search: this.dosearch
        };
      },
      skip() {
        return !this.dosearch;
      }
    }
  },
  data: () => ({
    currentCourse: null,
    active: [],
    loading: false,
    tree: [],
    model: null,
    search: null,
    dosearch: null
  }),
  computed: {
    selected() {
      if (!this.active.length) return undefined;

      const id = this.active[0];

      return id;
    },
    fields() {
      if (!this.model) return [];
      return Object.keys(this.model).map(key => {
        return {
          key,
          value: this.model[key] || "n/a"
        };
      });
    }
  },
  watch: {
    selected(newValue) {
      newValue ? (this.dosearch = newValue) : null;
    },
    $route: "fetchData",
    model: function(val) {
      if (val) {
        this.dosearch = val.code;
        this.fetchData(val.code);
      }
    }
  },
  created() {
    this.fetchData(this.$route.query.course);
  },
  methods: {
    fetchData(val) {
      this.loading = true;
      this.currentCourse = null;
      axios
        .get(`http://localhost:8080/api/tree?course=` + val)
        .then(response => {
          this.loading = false;
          this.currentCourse = val.toUpperCase();
          this.dosearch = val.toUpperCase();
          this.search = "";
          this.tree = [response.data];
        })
        .catch(e => {
          this.errors = e.toString();
        });
    }
  }
};
</script>
<style scoped>
.v-progress-circular {
  margin: 1rem;
}
</style>
