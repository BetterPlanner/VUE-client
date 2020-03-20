<template>
  <div class="Tree">
    <h1>Course Tree for {{ $route.query.course }}</h1>
    
    <tree v-if="!loading" :tree-data="tree"></tree>
  </div>
</template>

<script>
import axios from 'axios';
import Tree from '../components/Tree.vue'
export default {
  data () {
    return {
      loading: false,
      posts: null,
      errors: null,
      course: null,
      tree : null
    }
  },
  components: {
    Tree
  },
  created () {
    // fetch the data when the view is created and the data is
    // already being observed
    this.fetchData()
  },
  watch: {
    // call again the method if the route changes
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      this.errors = this.posts = null
      this.loading = true
      axios.get(`http://localhost:8080/tree1?course=`+this.$route.query.course, {crossDomain: true})
      .then(response => {
        this.loading = false
        // JSON responses are automatically parsed.
        this.tree = response.data
      })
      .catch(e => {
        this.errors = e.toString();
      })
    }
  },
  mounted: function() {
    if (this.$route.query.course) {
      this.course = this.$route.query.course;
    }
  }
}

</script>
