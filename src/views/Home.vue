<template>
  <div class="home">
    <header class="header">
      <input v-model="editText"
             type="text"
             class="edit"
             @keyup.enter="faddTodo()"
             placeholder="what needs to be done?"
             autofocus>
      <input id="toggle-all"
             type="checkbox"
             @click="ftoggleAll()">
      <label v-if="allCount"
             for="toggle-all"
             class="toggle-all"
             :class="{ 'all-done': completed.length === allCount }"></label>
    </header>
    <transition-group tag="ul"
                      class="todo-list"
                      appear>
      <li v-for="(item, index) in todos"
          :key="item.id"
          :class="{completed: item.completed, editing: index === editedTodoIndex}">
        <div class="view">
          <input class="toggle"
                 type="checkbox"
                 @click="ftoggleTodo(item, index)">
          <label @dblclick="feditTodo(item, index)">{{ item.text }}</label>
          <button class="destroy"
                  @click="fdeleteTodo(item, index)"></button>
        </div>
        <input v-model="tempText"
               v-todo-focus="index === editedTodoIndex"
               type="text"
               class="edit"
               @keyup.enter="fdoneEdit(item)"
               @keyup.esc="fcancelEdit()"
               @blur="fdoneEdit(item)">
      </li>
    </transition-group>
    <footer v-if="allCount"
            class="footer">
      <span class="count">{{ todos.length + (todos.length === 1 ? ' item' : ' items') }}</span>
      <ul class="filter">
        <li v-for="action in actions"
            :key="action.model">
          <span class="action"
                :class="{ active: action.model === model }"
                @click="fchangeModel(action.model)">{{ action.name }}</span></li>
      </ul>
      <span class="clear-btn"
            @click="fclearCompleted()">clear completed</span>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'home',
  created () {
    let active = localStorage.getItem('TODOLISTS_ACTIVE')
    let completed = localStorage.getItem('TODOLISTS_COMPLETED')
    if (active) {
      this.active = JSON.parse(active)
    }
    if (completed) {
      this.completed = JSON.parse(completed)
    }
  },
  data () {
    return {
      editText: '',
      tempText: '',
      editedTodoIndex: -1,
      active: [],
      completed: [],
      model: 'all',
      actions: [
        { name: 'all', model: 'all' },
        { name: 'active', model: 'active' },
        { name: 'completed', model: 'completed' }
      ]
    }
  },
  computed: {
    todos () {
      let model = this.model
      if (model === 'active') {
        return this.active
      } else if (model === 'completed') {
        return this.completed
      } else {
        return this.active.concat(this.completed)
      }
    },
    allCount () {
      return this.active.length + this.completed.length
    }
  },
  watch: {
    'active' (val) {
      localStorage.setItem('TODOLISTS_ACTIVE', JSON.stringify(val))
    },
    'completed' (val) {
      localStorage.setItem('TODOLISTS_COMPLETED', JSON.stringify(val))
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
      let text = this.editText.trim()
      if (text && text !== '') {
        this.active.unshift({ completed: false, text: this.editText, id: this.allCount })
        this.editText = ''
      }
    },
    ftoggleAll () {
      if (this.completed.length === this.allCount) {
        this.completed.forEach(item => {
          item.completed = false
        })
        this.active = this.completed.concat(this.active)
        this.completed.splice(0)
      } else {
        this.active.forEach(item => {
          item.completed = true
        })
        this.completed = this.active.concat(this.completed)
        this.active.splice(0)
      }
    },
    ftoggleTodo (item, index) {
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
    feditTodo (item, index) {
      this.tempText = item.text
      this.editedTodoIndex = index
    },
    fdoneEdit (item) {
      let model = this.model
      let position = this.fgetPosition(item.completed, this.editedTodoIndex)
      let text = this.tempText.trim()
      if (text && text !== '') {
        if (model === 'active' || !item.completed) {
          this.active[position].text = this.tempText
        } else {
          this.completed[position].text = this.tempText
        }
      } else {
        this.fdeleteTodo(item, this.editedTodoIndex)
      }
      this.fcancelEdit()
    },
    fcancelEdit () {
      this.editedTodoIndex = -1
    },
    fdeleteTodo (item, index) {
      let model = this.model
      let position = this.fgetPosition(item.completed, index)
      if (model === 'active' || !item.completed) {
        this.active.splice(position, 1)
      } else {
        this.completed.splice(position, 1)
      }
    },
    fchangeModel (model) {
      if (this.model !== model) {
        this.model = model
      }
    },
    fclearCompleted () {
      this.completed.splice(0)
    }
  },
  directives: {
    'todo-focus': function (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~@/assets/css/home.scss";
</style>
