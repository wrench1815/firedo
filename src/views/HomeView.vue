<script setup lang="ts">
import {
  collection,
  addDoc,
  Timestamp,
  orderBy,
  query,
  updateDoc,
  deleteDoc,
  doc,
} from 'firebase/firestore';
import { useCollection } from 'vuefire';
import { firestore } from '@/firebase/db';

import { ref } from 'vue';

const task = ref('');
const addingTask = ref(false);
const taskList = useCollection(
  query(collection(firestore, 'todo'), orderBy('timestamp', 'desc'))
);

async function addTodo() {
  addingTask.value = true;

  return addDoc(collection(firestore, 'todo'), {
    task: task.value,
    timestamp: Timestamp.now(),
    completed: false,
  }).then(() => {
    task.value = '';
    addingTask.value = false;
  });
}

function markComplete(task: string) {
  updateDoc(doc(firestore, 'todo', task), {
    completed: true,
  });
}

function deleteTask(task: string) {
  deleteDoc(doc(firestore, 'todo', task));
}
</script>

<template>
  <main class="container px-4 mx-auto">
    <section class="flex justify-center my-4">
      <div class="card w-96 bg-base-100 shadow-xl">
        <div class="card-body">
          <h2 class="card-title justify-center">Firedo🔥</h2>
          <div
            class="card-actions gap-3 justify-center item-center transition-all"
          >
            <input
              type="text"
              placeholder="Type here"
              class="input input-ghost w-auto"
              v-model="task"
            />
            <button
              class="btn btn-primary self-center btn-circle btn-sm"
              :class="{
                hidden: task.length < 2,
                'loading btn-disabled': addingTask,
              }"
              @click="addTodo"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="currentColor"
                class="w-5 h-5"
                :class="{
                  hidden: addingTask,
                }"
              >
                <path
                  d="M10.75 4.75a.75.75 0 00-1.5 0v4.5h-4.5a.75.75 0 000 1.5h4.5v4.5a.75.75 0 001.5 0v-4.5h4.5a.75.75 0 000-1.5h-4.5v-4.5z"
                />
              </svg>
            </button>
          </div>
          <section>
            <TransitionGroup name="list" tag="div">
              <div v-for="task in taskList" class="my-2" :key="task.id">
                <div
                  class="alert shadow-lg"
                  :class="task.completed ? 'alert-success' : 'alert-warning'"
                >
                  <div class="flex flex-col items-start">
                    <span>{{ task.task }}</span>

                    <span
                      >on {{ task.timestamp.toDate().toLocaleString() }}</span
                    >
                  </div>
                  <div>
                    <!-- completed -->
                    <button
                      class="btn btn-circle btn-sm btn-success border border-base-100 hover:border-base-100"
                      @click="markComplete(task.id)"
                      v-if="!task.completed"
                    >
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                        class="w-5 h-5"
                      >
                        <path
                          fill-rule="evenodd"
                          d="M16.704 4.153a.75.75 0 01.143 1.052l-8 10.5a.75.75 0 01-1.127.075l-4.5-4.5a.75.75 0 011.06-1.06l3.894 3.893 7.48-9.817a.75.75 0 011.05-.143z"
                          clip-rule="evenodd"
                        />
                      </svg>
                    </button>

                    <!-- deleted -->
                    <button
                      class="btn btn-circle btn-sm btn-error border border-base-100 hover:border-base-100"
                      @click="deleteTask(task.id)"
                    >
                      <svg
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                        fill="currentColor"
                        class="w-5 h-5"
                      >
                        <path
                          fill-rule="evenodd"
                          d="M8.75 1A2.75 2.75 0 006 3.75v.443c-.795.077-1.584.176-2.365.298a.75.75 0 10.23 1.482l.149-.022.841 10.518A2.75 2.75 0 007.596 19h4.807a2.75 2.75 0 002.742-2.53l.841-10.52.149.023a.75.75 0 00.23-1.482A41.03 41.03 0 0014 4.193V3.75A2.75 2.75 0 0011.25 1h-2.5zM10 4c.84 0 1.673.025 2.5.075V3.75c0-.69-.56-1.25-1.25-1.25h-2.5c-.69 0-1.25.56-1.25 1.25v.325C8.327 4.025 9.16 4 10 4zM8.58 7.72a.75.75 0 00-1.5.06l.3 7.5a.75.75 0 101.5-.06l-.3-7.5zm4.34.06a.75.75 0 10-1.5-.06l-.3 7.5a.75.75 0 101.5.06l.3-7.5z"
                          clip-rule="evenodd"
                        />
                      </svg>
                    </button>
                  </div>
                </div>
              </div>
            </TransitionGroup>
          </section>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>
