<template>
    <div
      v-show="isEmpty ? true : visible"
      id="draggable-node"
      :class="[isEmpty ? 'empty' : 'node', { active: isActive }, { 'appeared-empty': isShowEmpty }]"
      :draggable="!isEmpty"
      @dragover.prevent.stop="onDragover"
      @dragstart.stop="onDragstart"
      @dragenter.stop="onDragenter"
      @dragleave.stop="onDragleave"
      @dragend.stop="onDragend"
      @drop.stop="onDrop"
      >
      {{ data.name }}
        <div
          v-if="data.children && data.children.length"
          class="node-children">
          <draggable-node
            v-for="(child, index) in children"
            :key="index"
            :data="child"
            :index="index"
            :state="state"
            :options="options"
            >
            {{ child.name }}
          </draggable-node>
        </div>
    </div>
</template>

<script>
  export default {
    name: 'draggable-node',

    props: {
      options: {
        type: Object,
        default: () => ({
          nest: false
        })
      },

      data: {
        type: Object,
        required: true
      },

      state: {
        type: Object,
        default: () => ({
          draggingElm: null,
          passingElm: null
        })
      },

      index: Number
    },

    data: () => ({
      visible: true,
      isActive: false,
      showParentIndex: null
    }),

    computed: {
      children() {
        let { children } = this.data
        if (!children || !children.length) return []
        return children.reduce((prev, next) => [...prev, next, {}], [{}])
      },

      isShowEmpty() {
        if (!this.isEmpty) return false

        const parent = this.$parent
        const { draggingElm, passingElm } = this.state
        if (!passingElm) return false
        if (parent !== passingElm.$parent) return false
        if (this.options.nest) {
          if (Math.abs(this.index - passingElm.index) === 1) return true
        } else {
          if (draggingElm.$parent !== parent) return false
          if (this.index - passingElm.index === passingElm.showParentIndex) return true
        }
        return false
      },

      isEmpty() {
        return !this.data.name
      },

      isMeOrMyAncestor() {
        var self = this
        while (self) {
          if (this.state.draggingElm === self) return true
          self = self.$parent
        }
      },

      isAllowDrop() {
        if (!this.isMeOrMyAncestor) {
          return this.options.nest ? true : this.isEmpty
        }
        return false
      }
    },

    methods: {
      activeStyle() {
        this.isActive = true
      },

      clearStyle() {
        this.isActive = false
      },

      onDragstart() {
        this.state.draggingElm = this
        this.visible = false
      },

      onDragenter() {
        if (!this.isEmpty) {
          this.state.passingElm = this
        }
        if (!this.isAllowDrop) return
        this.activeStyle()
      },

      onDragover(e) {
        if (this.isEmpty) return

        const rect = this.$el.getBoundingClientRect()
        const height = rect.height
        const center = rect.top + height / 2 + document.body.scrollTop
        const eventY = e.pageY
        this.showParentIndex = eventY > center ? 1 : -1
      },

      onDragleave() {
        this.clearStyle()
      },

      onDragend() {
        this.visible = true
        this.state.draggingElm = this.state.passingElm = null
      },

      onDrop() {
        this.clearStyle()
        if (!this.isAllowDrop) return

        const draggingElm = this.state.draggingElm
        const draggingElmParent = draggingElm.$parent.data
        draggingElmParent.children.splice(draggingElmParent.children.indexOf(draggingElm.data), 1)

        if (this.isEmpty) {
          this.$parent.data.children.splice(this.index / 2, 0, draggingElm.data)
        }

        if (!this.data.children) this.$set(this.data, 'children', [])
        this.data.children.unshift(draggingElm.data)
      }
    }
  }
</script>

<style>
  #draggable-node {
    padding-left: 2em;
  }

  #draggable-node.empty {
    background-color: transparent;
    height: 0;
    transition: .2s;
  }

  #draggable-node.node.active, #draggable-node.empty.active {
    background-color: rgb(227, 246, 255);
  }

  #draggable-node.empty.appeared-empty {
    height: 40px;
  }
</style>
