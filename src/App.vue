<template>
  <div class="container">
    <h2>To-Do List</h2>
    <input
      class="form-control"
      type="text"
      v-model="searchText"
      placeholder="Search"
      @keyup.enter="searchTodo"
    >
    <hr />
    <TodoSimpleForm @add-todo="addTodo" />
    <div style="color: red">{{ error }}</div>
    
    <div v-if="!todos.length">
      Thers is nothing to display
    </div>
    <TodoList
      :todos="todos"
      @toggle-todo="toggleTodo"
      @delete-todo="deleteTodo"
    />
    <hr />
    <TodoPage
      :pageInfo="filteredPage"
      @get-todo="getTodos"
    />
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue';
import TodoSimpleForm from './components/TodoSimpleForm.vue';
import TodoList from './components/TodoList.vue'
import TodoPage from './components/TodoPage.vue'
import axios from 'axios';

export default{
  components: {
    TodoSimpleForm,
    TodoList,
    TodoPage,
  },
  setup() {
    const todos = ref([]);
    const error = ref('');
    const numberOfTodos = ref(0);
    const limit = 5;
    const currentPage = ref(1);
    const searchText = ref('');
    const numberOfPages = computed(() => {
      return Math.ceil(numberOfTodos.value / limit);
    });

    const todoStyle = {
      textDecoration: 'line-through',
      color: 'gray'
    }

    const getTodos = async (page = currentPage.value) => {
      currentPage.value = page;
      try {
        const res = await axios.get(`http://localhost:3000/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${limit}`);
        numberOfTodos.value = res.headers['x-total-count'];
        todos.value = res.data;
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong.';
      }
    };

    getTodos();

    const addTodo = async (todo) => {
      // 데이터베이스 투두를 저장
      error.value = '';
      try {
        await axios.post('http://localhost:3000/todos', {
          subject: todo.subject,
          completed: todo.completed,
        });
        
        getTodos(1);
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong.';
      }
    };

    const toggleTodo = async (index) => {
      error.value = '';
      const id = todos.value[index].id;
      try {
        await axios.patch('http://localhost:3000/todos/' + id, {
          completed: !todos.value[index].completed
        });
        todos.value[index].completed = !todos.value[index].completed;
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong.';
      }
    };

    const deleteTodo = async (index) => {
      error.value = '';
      const id = todos.value[index].id;
      try {
        await axios.delete('http://localhost:3000/todos/' + id);
        getTodos(1);
      } catch (err) {
        console.log(err);
        error.value = 'Something went wrong.';
      }    
    };

    let timeout = null;
    const searchTodo = () => {
      clearTimeout(timeout);
      getTodos(1);
    };

    watch(searchText, () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        getTodos(1);
      }, 1000);
    });
    // const filteredTodos = computed(() => {
    //   if (searchText.value) {
    //     return todos.value.filter(todo => {
    //       return todo.subject.includes(searchText.value);
    //     });
    //   }

    //   return todos.value; 
    // });

    const filteredPage = computed(() => {
      return {
        currentPage,
        numberOfPages,
      }
    });

    return {
      todos,
      addTodo,
      todoStyle,
      deleteTodo,
      toggleTodo,
      searchText,
      // filteredTodos,
      error,
      numberOfPages,
      currentPage,
      getTodos,
      filteredPage,
    };
  }
}
</script>

<style>
  .todo {
    color: gray;
    text-decoration: line-through;
  }
</style>