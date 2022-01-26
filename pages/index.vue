<template>
  <div>
    <h1>TodoApp</h1>
    <v-text-field v-model="name" label="Name"></v-text-field>
    <v-text-field v-model="description" label="Description"></v-text-field>
    <v-btn @click="createTodo">タスク生成する</v-btn>
    <ul>
      <li v-for="(todo, index) in todos" :key="todo.id">
        {{ todo.name }} : {{ todo.description }}
        <v-btn @click="deleteTodo(index, todo.id)">削除</v-btn>
        <div v-if="editflag == false">
          <v-btn @click="changeflag()">編集</v-btn>
        </div>
        <div v-else>
          <v-text-field type="text" v-model="todo.name" />
          <v-text-field type="text" v-model="todo.description" />
          <v-btn @click="updateTodo(todo.id, todo.name, todo.description)"
            >更新</v-btn
          >
        </div>
        <!-- <pre>{{ todo }}</pre> -->
      </li>
    </ul>
    <!-- <Tasklist></Tasklist> -->
    <!-- <v-btn @click="changeStore('hoge')">storeにアクセス</v-btn>
    <p>{{ $store.state.name }}</p> -->
  </div>
</template>

<script>
import { API } from 'aws-amplify'
import { createTodo, deleteTodo, updateTodo } from '~/src/graphql/mutations'
import { listTodos } from '~/src/graphql/queries'

export default {
  data() {
    return {
      name: '',
      description: '',
      todos: [],
      editflag: false,
    }
  },
  async mounted() {
    await this.getTodos()
  },
  methods: {
    changeStore(name) {
      this.$store.commit('changeName', name)
      //mutationsはstateの値を手軽に書き換えられる
      //commitはmutationの関数を起動する
      //dispatchはどのような時に使えるのか？？
      //this.$store.dispatch('changeStore', 'hoge');
    },
    changeflag() {
      this.editflag = true
    },
    async createTodo() {
      const { name, description } = this
      if (!name || !description) return false //空追加を防ぐためである
      const todo = { name, description }
      await API.graphql({
        query: createTodo,
        variables: { input: todo },
      })
      //初期化している
      this.name = ''
      this.description = ''
      this.getTodos()
    },
    async deleteTodo(index, todoId) {
      //todoIdはtodo.idと同じである
      await API.graphql({
        query: deleteTodo,
        variables: { input: { id: todoId } }, //ここがあまり何をしているかが理解できない
      })
        .then((result) => {
          //成功したらというやつやね
          //console.log(result);
          this.todos.splice(index, 1)
          //console.log(todoId);
          console.log(result.data.deleteTodo.name)
          console.log(result.data.deleteTodo.description)
          this.getTodos() //こいつをかますことで、しっかりと再初期化できる
        })
        //失敗したらというやつやね
        .catch((error) => {
          console.log(error)
        })
    },
    async updateTodo(todoId, name, description) {
      // const { name, description } = this
      //console.log(this)
      await API.graphql({
        query: updateTodo,
        variables: {
          input: {
            id: todoId,
            name: name,
            description: description,
          },
        },
        //ここがあまり何をしているかが理解できない
      })
        .then((result) => {
          //成功したらというやつやね
          console.log(result)
          this.editflag = false
          this.getTodos()
        })
        //失敗したらというやつやね
        .catch((error) => {
          console.log(error)
        })
    },
    async getTodos() {
      const todos = await API.graphql({
        query: listTodos,
      })
      this.todos = todos.data.listTodos.items
    },
  },
}
</script>
