<template>
  <div id="app" className="container">
    <div className="calendar-container">
      <WeeklyCalendar
          :tasks="tasks"
          @task-dropped="onTaskDropped"
          :dragged-task="draggedTask"
      />
    </div>
    <div className="task-container">
      <TaskList
          :tasks="tasks"
          @task-added="onTaskAdded"
          @task-dragged="onTaskDragged"
      />
    </div>
  </div>
</template>

<script>
import WeeklyCalendar from './components/WeeklyCalendar.vue';
import TaskList from './components/TaskList.vue';
import {ref} from 'vue';

export default {
  name: 'App',
  components: {
    WeeklyCalendar,
    TaskList
  },
  setup() {
    const tasks = ref([]);
    const draggedTask = ref(null);

    const onTaskAdded = (task) => {
      tasks.value.push(task);
    };

    const onTaskDragged = (task) => {
      // Store the dragged task
      draggedTask.value = task;
    };

    const onTaskDropped = (task, date) => {
      const index = tasks.value.findIndex(t => t.id === task.id);
      if (index !== -1) {
        tasks.value[index].date = date; // Update the task's date
        draggedTask.value = null; // Clear the dragged task
      }
    };

    return {
      tasks,
      draggedTask,
      onTaskAdded,
      onTaskDragged,
      onTaskDropped
    };
  }
};
</script>

<style>
.container {
  display: flex;
}

.calendar-container {
  flex: 3;
}

.task-container {
  flex: 1;
  padding: 20px;
}
</style>
