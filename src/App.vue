<script setup>
import { onMounted, ref } from 'vue';

const tasksModel = ref({
  tasks: [],
  editing: null
})

// Function to capitalize the first letter of a string
const capitalizeFirstLetter = (string) => {
  return string.charAt(0).toUpperCase() + string.slice(1);
};

// Function to format the date
const getDayOfWeek = () => {
  const date = new Date();
  return capitalizeFirstLetter(date.toLocaleDateString('fr-FR', { weekday: 'long' }));
};

const getDayOfMonth = () => {
  const date = new Date();
  return date.getDate();
};

const getMonth = () => {
  const date = new Date();
  return capitalizeFirstLetter(date.toLocaleDateString('fr-FR', { month: 'long' }));
};

const dayOfWeek = ref('');
const dayOfMonth = ref('');
const month = ref('');

onMounted(() => {
  dayOfWeek.value = getDayOfWeek();
  dayOfMonth.value = getDayOfMonth();
  month.value = getMonth();
});

const createTask = (data, form$) => {
  addToStorage(form$.data)
  syncFromStorage()

  form$.reset() // Pour remettre le formulaire à zero (valeurs par défaut)
  // form$.clear() // Pour vider le formulaire
}

const addToStorage = (data) => {
  let storageData = localStorage.getItem("tasks")
  storageData = storageData ? JSON.parse(storageData) : []

  storageData.push({ ...data, completed: false }) // Ajout de la propriété completed
  localStorage.setItem('tasks', JSON.stringify(storageData))
}

const syncFromStorage = () => {
  let tasks = localStorage.getItem("tasks")

  tasksModel.value = {
    tasks: tasks ? JSON.parse(tasks) : []
  }
}

const syncToStorage = () => {
  localStorage.setItem('tasks', JSON.stringify(tasksModel.value.tasks))
}

const toggleTaskCompletion = (index) => {
  tasksModel.value.tasks[index].completed = !tasksModel.value.tasks[index].completed
  syncToStorage()
}

const edit = (index) => {
  tasksModel.value.editing = index
}

const cancel = (index) => {
  tasksModel.value.editing = null
  syncFromStorage()
}

const save = () => {
  syncToStorage()
  tasksModel.value.editing = null
}

onMounted(() => {
  syncFromStorage()
})
</script>

<template>
  <div class="page">
    <div class="container">
      <h1>{{ dayOfWeek }}, {{ dayOfMonth }}</h1>
      <h2>{{ month }}</h2>

      <Vueform size="lg" :endpoint="createTask">
        <TextElement 
            name="task" 
            placeholder="Ajouter une tâche" 
            floating="Nom de la tâche" 
            rules="required" 
        />

      </Vueform>

      <hr class="divider" />

      <Vueform v-model="tasksModel" sync>
        <ListElement
            name="tasks"
            :controls="{
                add: false,
            }"
        >
        <template #default="{ index }">
            <ObjectElement
                :name="index"
                :add-class="['task-container']">
              
                <div class="task-wrapper">
                <span :class="{
                    'task-label': true, 
                    'task-completed': tasksModel.tasks[index].completed
                  }">
                  {{ tasksModel.tasks[index].task }}
                </span>
                <div class="circle" @click="toggleTaskCompletion(index)"></div>
              </div>
            </ObjectElement>
        </template>
      </ListElement>

      <HiddenElement name="editing" />

      </Vueform>
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.page {
  background: linear-gradient(0deg, rgb(0, 118, 245) 0%, rgba(51,170,255,1) 100%);
  width: 100%;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  width: 550px;
  margin: 2%;
  background: white;
  padding: 2%;
  border-radius: 10px;
}

h1 {
  font-size: 2em;
  margin-bottom: 0.1em;
  font-weight: 700;
  font-family: "Nunito", sans-serif;
  color: #0e36ca;
}

h2 {
  font-size: 1.5em;
  font-weight: 600;
  font-family: "Nunito", sans-serif;
  color: #0e37caa5;
}

.divider {
  margin: 2rem 0;
  border-color: #e2e8f0;
}

.task-container {
  background: #ffffff;
  padding: 0.5em;
}

.task-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.task-label {
  flex-grow: 1;
  margin-right: 1rem;
  word-break: break-word;
  transition: color 0.3s ease, text-decoration 0.3s ease;
}

.task-completed {
  color: rgb(119, 119, 119);
  text-decoration: line-through;
}

.circle {
  width: 20px;
  height: 20px;
  background-color: white;
  border: solid 2px #ADB0AB;
  border-radius: 50%;
  cursor: pointer;
}

.vf-row {
  display: block !important; 
}

.vf-list-remove span, .vf-list-handle span {
  display: none !important;
}

</style>


