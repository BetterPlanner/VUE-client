<template>
  <!-- <li class="node-tree">
    <v-row><span class="label">{{ node.label }}</span></v-row>
    <ul v-if="node.nodes && node.nodes.length">
      <node v-for="(child, idx) in node.nodes" :node="child" :key="idx"></node>
    </ul>
  </li> -->

  <v-container>
    <v-row><span class="label">{{ node.label }}</span></v-row>
    <v-row no-gutters v-if="node.nodes && node.nodes.length">
      <template>
        <v-col :key="idx">
          <v-card
            class="pa-2"
            outlined
            tile
          >
            <node v-for="(child, idx) in node.nodes" :node="child" :key="idx"></node>
          </v-card>
        </v-col>
      </template>
    </v-row>
  </v-container>
</template>
<script>
  export default { 
    props: {
      node: Object
    },
    name: 'node'
  }
</script>
