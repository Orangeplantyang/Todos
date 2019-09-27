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
          <label :id="item.id"
                 :draggable="model!=='all'"
                 @dragstart="fonDragStart"
                 @dragover="fonDragOver"
                 @dragend="fonDragEnd"
                 @dblclick="feditTodo(item, index)">{{ item.text }}</label>
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
  // mounted () {
  //   document.body.ondrop = function (event) {
  //     event.preventDefault()
  //     event.stopPropagation()
  //   }
  // },
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
      ],
      draging: null,
      target: null // 目标对象
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
    fonDragStart (event) {
      this.draging = event.target
    },
    fonDragOver (event) {
      event.preventDefault()
      if (event.target !== this.draging) {
        this.target = event.target
      }
    },
    fonDragEnd (event) {
      if (this.target && event.target !== this.target) {
        this.fexchange(this.draging.id, this.target.id)
      }
    },
    fexchange (id1, id2) {
      let index1 = 0
      let index2 = 0
      let array = []
      if (this.model === 'active') {
        array = this.active
      } else {
        array = this.completed
      }
      for (let i = 0; i < array.length; i++) {
        if (id1.toString() === array[i].id.toString()) {
          index1 = i
        }
        if (id2.toString() === array[i].id.toString()) {
          index2 = i
        }
      }
      this.$nextTick(() => {
        // [array[index1], array[index2]] = [array[index2], array[index1]]
        array.splice(index2, 1, ...array.splice(index1, 1, array[index2]))
      })
    },
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
        let id = (Math.random().toString().substr(3, 27) + Date.now()).toString(24)
        this.active.unshift({ completed: false, text: this.editText, id: id })
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
