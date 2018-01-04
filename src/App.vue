<template>
  <div id="app">
    <draggable
      :data="data">
    </draggable>
  </div>
</template>

<script>
import draggable from './draggable.vue'

export default {
  name: 'app',

  components: {
    draggable
  },

  data: () => ({
    data: {
      name: 'root',
      children: [
        {
          name: '1-1'
        },
        {
          name: '1-2',
          children: [
            {
              name: '2-1'
            },
            {
              name: '2-2'
            },
            {
              name: '2-3'
            },
            {
              name: '2-4'
            }
          ]
        },
        {
          name: '1-3'
        }
      ]
    }
  })
}
</script>

<style>
  body {
    background-color: #eee;
  }

  #app {
    width: 500px;
    margin: 0 auto;
    line-height: 40px;
    border: 1px solid #ddd;
    background-color: #fff;
  }
</style>
