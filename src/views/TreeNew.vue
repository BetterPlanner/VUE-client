<template>
  <div class="Tree">
    <courseTree></courseTree>
  </div>
</template>
<script>
import CourseTree from "../components/CourseTree.vue";
export default {
  data() {
    return {
      loading: false,
      posts: null,
      errors: null,
      course: null,
      tree: null
    };
  },
  components: {
    CourseTree
  }
};
</script>
