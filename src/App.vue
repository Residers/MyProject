<template>
  <v-app>
    <div class="app">
      <v-card-title class="app__title">TODOS</v-card-title>
      <div class="app__btn">
        <v-btn variant="outlined" @click="setFilterCompleted">All</v-btn>
        <v-btn variant="outlined" @click="setFilterCompleted('active')"
          >Active</v-btn
        >
        <v-btn variant="outlined" @click="setFilterCompleted('done')"
          >Done</v-btn
        >
        <v-btn
          @click="isDialog = true"
          icon="mdi-android-messages"
          variant="outlined"
        />
      </div>
      <MyDialog v-model="isDialog" @saveTask="saveTask" />
      <TodoList
        :tasks="tasks"
        :filterType="filterType"
        @editTask="editTask"
        @toggleTask="toggleTask"
        @removeTask="removeTask"
      />
    </div>
  </v-app>
</template>

<script setup lang="ts">
// import { isRequired } from "./helpers/rules";
import { ref, onMounted, provide } from "vue";
import TodoList from "./components/list/TodoList.vue";
import MyDialog from "./components/dialog/MyDialog.vue";
import type { Task } from "@/type";

let id = 0;
const isDialog = ref(false);
const isEditTask = ref(false);
const tasks = ref([] as Task[]);
const taskTitle = ref("");
const filterType = ref<"all" | "active" | "done">("all");
const currenTaskId = ref(null);

onMounted(() => {
  const storedTasks = localStorage.getItem("tasks");
  if (storedTasks) {
    tasks.value = JSON.parse(storedTasks);
  }
});

function addTask() {
  if (taskTitle.value) {
    tasks.value.unshift({
      title: taskTitle.value,
      complete: false,
      id: id++,
    });
    taskTitle.value = "";
    isDialog.value = false;
  }
}

function editTask(task) {
  taskTitle.value = task.title;
  currenTaskId.value = task.id;
  isEditTask.value = true;
  isDialog.value = true;
}
function saveTask() {
  if (isEditTask.value && currenTaskId.value !== null) {
    const task = tasks.value.find((t) => t.id === currenTaskId.value);
    if (task) {
      task.title = taskTitle.value;
    }
  } else addTask();
  closeForm();
  localStorage.setItem("tasks", JSON.stringify(tasks.value));
}

function closeForm() {
  isDialog.value = false;
  isEditTask.value = false;
  taskTitle.value = "";
  currenTaskId.value = null;
}

function toggleTask(taskId: number) {
  const task = tasks.value.find((task) => task.id === taskId);
  if (task) {
    task.complete = !task.complete;
    localStorage.setItem("tasks", JSON.stringify(tasks.value));
  }
}

function removeTask(taskId: number) {
  tasks.value = tasks.value.filter((task) => task.id !== taskId);
  localStorage.setItem("tasks", JSON.stringify(tasks.value));
}
function setFilterCompleted(type: "active" | "done") {
  filterType.value = type;
}
</script>

<style lang="scss" scoped>
.completeTask {
  text-decoration: line-through;
}

.app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: rgb(128, 17, 161);
  &__title {
    font-size: 30px;
  }
  &__btn {
    margin-bottom: 15px;
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 30px;
  }
}

.card {
  width: 400px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border-top: 1px solid;
  padding: 10px;

  &__list {
    display: flex;
    flex-direction: row;
  }
  &__items {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    gap: 10px;
    color: rgb(128, 17, 161);
    &:hover {
      .card__btn {
        opacity: 1;
      }
    }
  }
  &__text {
    word-wrap: break-word;
    width: 300px;
    font-size: 16px;
  }
  &__btns {
    display: flex;
    gap: 10px;
  }
  &__btn {
    width: 30px;
    height: 30px;
    opacity: 0;
  }
}
.v-card-text {
  padding: 5px;
}

.item-list {
  width: 100%;
  border: 1px solid;
  border-radius: 15px;
  padding: 15px;
  gap: 20px;
  color: rgba(127, 17, 161);
  box-shadow: 1px 1px 10px rgba(127, 17, 161, 0.555);

  &__task {
    flex-grow: 1;
  }

  &__btn {
    width: 100%;
    margin-top: 10px;
  }
  &__btn-close {
    color: rgba(127, 17, 161);
    width: 20px;
    height: 20px;
    border: 1px solid;
    &:deep(.v-icon) {
      font-size: 16px;
    }
  }
}
.showAddTask-enter-active,
.showAddTask-leave-active {
  transition: opacity 3s ease;
}
.showAddTask-enter-from,
.showAddTask-leave-to {
  transition: opacity 0;
  transform: translateX(30px);
}
</style>
