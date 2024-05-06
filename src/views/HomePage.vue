<template>
  <div class="side-menu">
    <button @click="toggleDeleteMode" class="icon-button">
      <IconPark type="delete" theme="outline" size="20" fill="#333"/>
    </button>
    <button @click="showAllTasks" class="icon-button">
      <IconPark type="view-list" theme="outline" size="24" fill="#333"/>
    </button>
    <button @click="showCompletedTasks" class="icon-button">
      <IconPark type="check-correct" theme="outline" size="24"/>
    </button>
    <button @click="showUncompletedTasks" class="icon-button">
      <IconPark type="checkbox" theme="outline" size="24" fill="#333"/>
    </button>
  </div>

  <div class="container">
    <div class="menu">
      <div class="folders">
        <div v-for="folder in folders" :key="folder.id" class="folder" @click="selectFolder(folder.id)"
             :class="{ 'active': funct(folder.id) }">
          {{ folder.name }}
          <span v-if="deleteMode" class="delete-icon" @click.stop="deleteFolder(folder.id)">×</span>
        </div>
        <div class="add-button" @click="adding ? null : toggleAdding(true)">
          <input v-if="adding" v-model="newFolderName" @keyup.enter="addFolder" placeholder="Новая папка" @blur="toggleAdding(false)" autofocus maxlength="20">
          <svg v-else viewBox="0 0 24 24" fill="white" xmlns="http://www.w3.org/2000/svg" width="16" height="16">
            <path d="M19 11h-6V5h-2v6H5v2h6v6h2v-6h6v-2z"/>
          </svg>
        </div>
      </div>
      <div>
        <button class="icon-button login">
          <IconPark type="people-plus-one" theme="outline" size="24"/>
        </button>
      </div>
    </div>
    <div class="content">
      <div class="tasks">
        <div class="tasks">
          <div v-for="task in selectedFolderTasks" :key="task.id" class="task" :class="{ 'completed': task.completed }">
            <input type="checkbox" v-model="task.completed" @change="() => updateTaskCompletion(task)">
            <span v-if="editingTaskId !== task.id" @dblclick="() => startEditTask(task)" style="padding-right: 5px">
              {{ task.text }}
            </span>
            <input v-else type="text" v-model="editingTaskText" @keyup.enter="() => saveTask(task)" @blur="cancelEdit" class="task-input" style="all: unset; width: 100%;">
            <span v-if="deleteMode" class="delete-icon" @click.stop="deleteTask(task.id, selectedFolderId)">×</span>
          </div>
        </div>
      </div>

      <button @click="addingTask = !addingTask" class="add-task-button">Добавить запись</button>
      <input v-if="addingTask" v-model="newTaskText" @keyup.enter="() => addNewTask(selectedFolderId, newTaskText)" placeholder="Введите новую задачу" class="task-input" @blur="addingTask = false">
    </div>
  </div>
</template>




<script>
import { ref, onMounted } from 'vue';
import { IconPark } from '@icon-park/vue-next/es/all';

export default {
  setup() {
    const folders = ref([]);
    const selectedFolderId = ref(null);
    const selectedFolderTasks = ref([]);
    const adding = ref(false);
    const addingTask = ref(false);
    const newFolderName = ref("");
    const newTaskText = ref("");
    const editingTaskId = ref(null); // Добавляем состояние редактирования задачи
    const editingTaskText = ref(""); // Текст для редактируемой задачи
    const currentFilter = ref("all");
    const deleteMode = ref(false);

    const toggleDeleteMode = () => {
      deleteMode.value = !deleteMode.value;
    };

    const deleteFolder = (folderId) => {
      folders.value = folders.value.filter(folder => folder.id !== folderId);
      saveFoldersToStorage();
      if (selectedFolderId.value === folderId) {
        selectedFolderId.value = null;
        selectedFolderTasks.value = [];
      }
    };

    const deleteTask = (taskId, folderId) => {
      const folder = folders.value.find(f => f.id === folderId);
      if (folder) {
        folder.content = folder.content.filter(task => task.id !== taskId);
        saveFoldersToStorage();
        selectedFolderTasks.value = folder.content;
      }
    };

    const startEditTask = (task) => {
      editingTaskId.value = task.id;
      editingTaskText.value = task.text;
    };

    const saveTask = (task) => {
      const folder = folders.value.find(f => f.id === selectedFolderId.value);
      if (folder) {
        const taskToEdit = folder.content.find(t => t.id === task.id);
        if (taskToEdit) {
          taskToEdit.text = editingTaskText.value;
          saveFoldersToStorage();
        }
      }
      editingTaskId.value = null;
    };

    const cancelEdit = () => {
      editingTaskId.value = null;
    };

    const funct = (id) => {
      console.log(id);
      console.log(selectedFolderId.value);
      console.log(id === selectedFolderId.value);
      return id === selectedFolderId.value;
    }

    // Функции для работы с localStorage
    const loadFoldersFromStorage = () => {
      let storedFolders = localStorage.getItem('folders');
      if (storedFolders) {
        storedFolders = JSON.parse(storedFolders);
      } else {
        // Если в localStorage нет папок, создаем папку "Заметки" по умолчанию
        storedFolders = [
          {
            id: 0,
            name: "Заметки",
            content: []
          }
        ];
        localStorage.setItem('folders', JSON.stringify(storedFolders));
      }
      return storedFolders;
    };

    const saveFoldersToStorage = () => {
      localStorage.setItem('folders', JSON.stringify(folders.value));
    };

    const updateTaskCompletion = (task) => {
      const folder = folders.value.find(folder => folder.id === selectedFolderId.value);
      if (folder) {
        const taskToUpdate = folder.content.find(t => t.id === task.id);
        console.log(taskToUpdate.completed);
        if (taskToUpdate) {
          this.$set(taskToUpdate, 'completed', !taskToUpdate.completed);
        }
        // Переключаем состояние задачи
        saveFoldersToStorage(); // Сохраняем обновлённые данные
      }
    };


    const selectFolder = (folderId) => {

      selectedFolderId.value = folderId;
      const folderElement = document.querySelector('.folder.active');
      const slider = document.getElementById('slider');

      if (folderElement && slider) {
        slider.style.width = `${folderElement.offsetWidth}px`; // Устанавливаем ширину ползунка равной ширине элемента папки
        slider.style.transform = `translateX(${folderElement.offsetLeft}px)`; // Перемещаем ползунок к активной папке
      }

      const folder = folders.value.find(f => f.id === folderId);
      selectedFolderTasks.value = folder ? folder.content : [];
    };



    // Функция для добавления новой папки
    const addFolder = () => {
      if (newFolderName.value.trim() !== '') {
        let newId = folders.value.length > 0 ? Math.max(...folders.value.map(folder => folder.id)) + 1 : 1;
        const newFolder = {
          id: newId,
          name: newFolderName.value,
          content: []
        };
        folders.value.push(newFolder);
        saveFoldersToStorage();
        newFolderName.value = '';
        adding.value = false;
        selectFolder(newId);
      }
    };

    // Функция для добавления новой задачи
    const addNewTask = () => {
      if (newTaskText.value.trim() !== '' && selectedFolderId.value !== null) {
        const folder = folders.value.find(folder => folder.id === selectedFolderId.value);
        if (folder) {
          const newTask = {
            id: folder.content.length > 0 ? Math.max(...folder.content.map(task => task.id)) + 1 : 1,
            text: newTaskText.value,
            completed: false,
          };
          folder.content.push(newTask);
          saveFoldersToStorage();
          newTaskText.value = '';
          addingTask.value = false;
        }
      }
    };


    const showAllTasks = () => {
      currentFilter.value = "all";
      filterTasks();
    };

    const showCompletedTasks = () => {
      currentFilter.value = "completed";
      filterTasks();
    };

    const showUncompletedTasks = () => {
      currentFilter.value = "uncompleted";
      filterTasks();
    };

    const filterTasks = () => {
      const folder = folders.value.find(f => f.id === selectedFolderId.value);
      if (folder) {
        switch (currentFilter.value) {
          case 'all':
            selectedFolderTasks.value = folder.content;
            break;
          case 'completed':
            selectedFolderTasks.value = folder.content.filter(task => task.completed);
            break;
          case 'uncompleted':
            selectedFolderTasks.value = folder.content.filter(task => !task.completed);
            break;
        }
      }
    };

    const toggleAdding = (value) => {
      adding.value = value;  // Устанавливаем состояние добавления папки
    };

    onMounted(() => {
      folders.value = loadFoldersFromStorage();
      selectFolder(0);
      if (selectedFolderId.value) {
        const folder = folders.value.find(f => f.id === selectedFolderId.value);
        if (folder) {
          selectedFolderTasks.value = folder.content;
        }
      }
    });

    // Возвращаем все необходимые переменные и функции
    return {
      toggleDeleteMode,
      showAllTasks,
      showCompletedTasks,
      showUncompletedTasks,
      folders,
      selectFolder,
      toggleAdding,
      updateTaskCompletion,
      funct,
      startEditTask,
      saveTask,
      cancelEdit,
      deleteTask,
      deleteFolder,

      selectedFolderId,
      selectedFolderTasks,
      adding,
      addingTask,
      newFolderName,
      newTaskText,
      addFolder,
      addNewTask,
      editingTaskId,
      editingTaskText,
      filterTasks,
      deleteMode
    };
  },
  components: {
    IconPark
  }
}

</script>

<style>


body, html {
  margin: 0;
  padding: 0;
  background-color: #f0f0f0;
  font-family: 'Montserrat', sans-serif;
  font-size: 18px;
  height: 100%; /* Устанавливаем высоту всей страницы */
}

.side-menu {
  width: 40px;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
  background-color: #f0f0f0;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 10px;
  padding-top: 60px; /* Добавляем отступ сверху */
}

.icon-button {
  background: none;
  border: none;
  margin: 10px 0;
  cursor: pointer;
}

.icon-button i,
.icon-button svg {
  width: 24px;
  height: 24px;
}


.menu {
  height: 40px; /* Высота верхнего меню */
  background-color: #f0f0f0;
  position: fixed;
  top: 0;
  left: 40px; /* Начинается справа от бокового меню */
  right: 0;
  display: flex;
  margin: 10px;
  align-items: center;
  padding: 0 20px; /* Отступы внутри меню */
  z-index: 1000;
  justify-content: space-between;
}

.container {
  position: fixed; /* Фиксированное позиционирование */
  top: 0; /* Смещение сверху равно высоте верхнего меню */
  left: 0; /* Смещение слева равно ширине бокового меню */
  right: 0;
  bottom: 0;
  overflow: hidden; /* Добавляем прокрутку, если содержимое больше размера контейнера */
}

.content {
  color: black;
  margin-top: 20px; /* Отступ для контента внутри container для визуального разделения */
  height: 100%;
  width: 100%;
  background-color: white;
  border-radius: 10px 0 0 0;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2); /* Добавляем тень */
}


.folders, .add-button {
  display: flex;
  gap: 10px;
  cursor: pointer;
  margin: 5px;
}

.folder {
  color: gray;
  cursor: pointer;
  padding: 5px 10px;
  border-radius: 20px; /* Овальные скругления */
  transition: background-color 0.3s, padding 0.3s; /* Анимация изменений */
}

.active {
  background-color: #c4c4c4; /* Выделение активной папки */
}

/* Динамическое изменение размера на основе содержимого */
.folder.active {
  color: black;
  padding-left: 10px;
  padding-right: 10px;
}

.slider {
  height: 2px;
  background-color: #c4c4c4; /* Цвет ползунка */
  position: absolute; /* Позиционирование относительно ближайшего относительно позиционированного родителя */
  transition: transform 0.3s ease, width 0.3s ease; /* Анимация перемещения и изменения размера */
  bottom: 0; /* Расположение ползунка в нижней части элементов папки */
}



.add-button {
  width: 30px; /* Начальная ширина */
  height: 30px; /* Начальная высота */
  border-radius: 50%; /* Округлая форма */
  background-color: #f5b000; /* Желтый фон */
  display: flex;
  justify-content: center;
  align-items: center;
  transition: width 0.3s ease, background-color 0.3s ease; /* Анимация изменения ширины и цвета фона */
  cursor: pointer;
  overflow: hidden; /* Скрывает части дочерних элементов, выходящие за пределы родителя */
}

.add-button:hover {
  background-color: #ecc849; /* Светлее желтый при наведении */
}

.add-button input {
  width: 100%; /* Позволяет полю ввода занять всю ширину кнопки */
  height: 30px; /* Высота, равная высоте кнопки */
  border: none;
  border-radius: 50px; /* Увеличиваем радиус для более овальной формы при расширении */
  padding: 0 10px; /* Отступы внутри поля ввода */
  font-size: 16px;
  background-color: transparent; /* Прозрачный фон */
  color: black; /* Цвет текста */
  outline: none; /* Убирает контур при фокусе */
}

.add-button input:focus, .add-button input:not(:placeholder-shown) {
  width: 180px; /* Расширенная ширина */
  border-radius: 25px; /* Более овальная форма при расширении */
}

.add-button:focus-within {
  width: 180px; /* Увеличиваем ширину всей кнопки */
  background-color: white; /* Меняем цвет фона на белый */
  border-radius: 25px; /* Скругляем углы при расширении */
}



.content {
  margin-top: 60px; /* Отступ сверху для контента */
  margin-left: 60px;
  padding: 20px;
}

.tasks {
  margin-top: 20px;
}

.task {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  font-family: 'Montserrat', sans-serif;
  font-size: 20px;
}

.task input[type="checkbox"] {
  margin-right: 10px;
}

.task-input {
  margin-top: 10px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-family: 'Montserrat', sans-serif;
  font-size: 16px;
}


.task.completed {
  text-decoration: line-through;
  opacity: 0.5;
}

.add-task-button {
  background-color: #f5b000;
  color: white;
  border: none;
  border-radius: 30px;
  padding: 10px 20px;
  margin-top: 20px;
  cursor: pointer;
}

.add-task-button:hover {
  background-color: #ecc849;
}

.add-task-button:focus {
  outline: none;
}

.task span {
  user-select: none; /* Предотвращает выделение текста */
}

.delete-icon {
  margin-left: 0;
  cursor: pointer;
  color: #413e3e;
}

.login {
  right: 0;
  float: right;
}

</style>
