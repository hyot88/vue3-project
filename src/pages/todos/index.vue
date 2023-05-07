<template>
  <div>
    <div class="d-flex justify-content-between mb-3">
      <h2>To-Do List</h2>
      <button
        class="btn btn-primary"
        @click="moveToCreatePage"
      >
        Create Todo
      </button>
    </div>
    
    <input
      class="form-control"
      type="text"
      v-model="searchText"
      placeholder="Search"
      @keyup.enter="searchTodo"
    >
    <hr />
    
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
  <Toast
    v-if="showToast"
    :message="toastMessage"
    :type="toastAlertType"
  />
</template>

<script>
import { ref, computed, watch } from 'vue';
import TodoList from '@/components/TodoList.vue'
import TodoPage from '@/components/TodoPage.vue'
import axios from '@/axios';
import Toast from '@/components/Toast.vue';
import { useToast } from '@/composables/toast'
import { useRouter } from 'vue-router'

export default{
  components: {
    TodoList,
    TodoPage,
    Toast,
  },
  setup() {
    const router = useRouter();
    const todos = ref([]);
    const error = ref('');
    const numberOfTodos = ref(0);
    const limit = 5;
    const currentPage = ref(1);
    const searchText = ref('');
    const numberOfPages = computed(() => {
      return Math.ceil(numberOfTodos.value / limit);
    });
    const {showToast,
        toastMessage,
        toastAlertType,
        triggerToast
    } = useToast();

    const todoStyle = {
      textDecoration: 'line-through',
      color: 'gray'
    }

    const getTodos = async (page = currentPage.value) => {
      currentPage.value = page;
      try {
        const res = await axios.get(`todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${limit}`);
        numberOfTodos.value = res.headers['x-total-count'];
        todos.value = res.data;
      } catch (err) {
        console.log(err);
        triggerToast('Something went wrong', 'danger');
      }
    };

    getTodos();

    const addTodo = async (todo) => {
      // 데이터베이스 투두를 저장
      error.value = '';
      try {
        await axios.post('todos', {
          subject: todo.subject,
          completed: todo.completed,
        });
        
        getTodos(1);
      } catch (err) {
        console.log(err);
        triggerToast('Something went wrong', 'danger');
      }
    };

    const toggleTodo = async (index, checked) => {
      error.value = '';
      const id = todos.value[index].id;
      try {
        await axios.patch('todos/' + id, {
          completed: checked
        });
        todos.value[index].completed = checked;
      } catch (err) {
        console.log(err);
        triggerToast('Something went wrong', 'danger');
      }
    };

    const deleteTodo = async (id) => {
      error.value = '';
      try {
        await axios.delete('todos/' + id);
        getTodos(1);
      } catch (err) {
        console.log(err);
        triggerToast('Something went wrong', 'danger');
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

    const filteredPage = computed(() => {
      return {
        currentPage,
        numberOfPages,
      }
    });

    const moveToCreatePage = () => {
      router.push({
        name: 'TodoCreate'
      });
    };

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
      searchTodo,
      toastMessage,
      toastAlertType,
      showToast,
      moveToCreatePage,
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