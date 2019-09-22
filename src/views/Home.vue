<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <input v-model="editText"
           type="text"
           class="edit"
           @keyup.enter="faddTodo()">
    <transition-group tag="ul"
                      class="todo-list"
                      appear>
      <li v-for="(item, index) in shows"
          :key="item"
          :class="{completed: item.completed, editing: index === editedTodoIndex}">
        <div class="view">
          <input class="toggle"
                 type="checkbox"
                 @click="fcheckTodo(item, index)">
          <label @dblclick="feditTodo(item, index)">{{ item.text }}</label>
          <button class="destroy"
                  @click="fdeleteTodo(item, index)"></button>
        </div>
        <!-- <input v-model="editText"
              type="text"
              class="edit"
              @keyup.enter="feditTodo()"> -->
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  name: 'home',
  data () {
    return {
      editText: '',
      editedTodoIndex: 0,
      active: [],
      completed: [],
      model: 'all'
    }
  },
  computed: {
    shows () {
      let model = this.model
      if (model === 'active') {
        return this.active
      } else if (model === 'completed') {
        return this.completed
      } else {
        return this.active.concat(this.completed)
      }
    }
  },
  methods: {
    fgetPosition (completed, index) {
      if (this.model === 'active' || this.model === 'completed' || !completed) {
        return index
      } else {
        return index - this.active.length
      }
    },
    faddTodo () {
      this.active.unshift({ completed: false, text: this.editText })
    },
    fcheckTodo (item, index) {
      let model = this.model
      let position = this.fgetPosition(item.completed, index)
      if (model === 'active' || !item.completed) {
        let obj = this.active[position]
        this.active.splice(position, 1)
        obj.completed = true
        this.$nextTick(() => {
          this.completed.unshift(obj)
        })
      } else {
        let obj = this.completed[position]
        this.completed.splice(position, 1)
        obj.completed = false
        this.$nextTick(() => {
          this.active.unshift(obj)
        })
      }
    },
    fdeleteTodo (item, index) {
      let model = this.model
      let position = this.fgetPosition(item.completed, index)
      if (model === 'active' || !item.completed) {
        this.active.splice(position, 1)
      } else {
        this.completed.splice(position, 1)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~@/assets/css/home.scss";
</style>
