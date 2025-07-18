<script setup lang="ts">
import '@/assets/main.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '../../amplify/data/resource';
import { generateClient } from 'aws-amplify/data';

const client = generateClient<Schema>();

// create a reactive reference to the array of todos
const todos = ref<Array<Schema['Todo']["type"]>>([]);

function listTodos() {
  client.models.Todo.observeQuery().subscribe({
    next: ({ items }) => {
      todos.value = [...items].sort((a, b) => {
        const dateA = new Date(a.eventDate || '');
        const dateB = new Date(b.eventDate || '');
        return dateA.getTime() - dateB.getTime(); // ascending
      });
    },
  });
}

function createTodo() {
  client.models.Todo.create({
    content: window.prompt("Todo content"),
    eventDate: window.prompt("Event date (YYYY-MM-DD)"),
  }).then(() => {
    // After creating a new todo, update the list of todos
    listTodos();
  });
}

// fetch todos when the component is mounted
onMounted(() => {
  listTodos();
});

</script>

<template>
  <main>
    <h1>My todos</h1>
    <button @click="createTodo">+ new</button>
    <ul>
      <li v-for="todo in todos"
          :key="todo.id">
        {{ todo.content }}, {{ todo.eventDate ? new Date(todo.eventDate).toLocaleDateString() : 'No date' }}
      </li>
    </ul>
    <div>
      🥳 App successfully hosted. Try creating a new todo.
      <br />
      <a href="https://docs.amplify.aws/gen2/start/quickstart/nextjs-pages-router/">
        Review next steps of this tutorial.
      </a>
    </div>
  </main>
</template>
