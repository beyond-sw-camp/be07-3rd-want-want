<template>
  <div class="task-list">
    <div
        v-for="task in tasks"
        :key="task.id"
        class="task-item"
        :draggable="true"
        @dragstart="onDragStart(task)"
    >
      {{ task.name }}
    </div>
    <button @click="addTask">+ Add Task</button>
  </div>
</template>

<script>
import { ref } from 'vue';
import { v4 as uuidv4 } from 'uuid';

export default {
  name: 'TaskList',
  props: {
    tasks: Array
  },
  emits: ['task-added', 'task-dragged'],
  setup(props, { emit }) {
    const taskId = ref(0);

    const addTask = () => {
      const newTask = {
        id: uuidv4(),
        name: `Task ${taskId.value + 1}`,
        date: null
      };
      emit('task-added', newTask);
      taskId.value++;
    };

    const onDragStart = (task) => {
      // Emit the dragged task
      emit('task-dragged', task);
    };

    return {
      addTask,
      onDragStart
    };
  }
};
</script>

<style>
.task-list {
  display: flex;
  flex-direction: column;
}
.task-item {
  padding: 10px;
  border: 1px solid #ccc;
  margin-bottom: 5px;
  cursor: grab;
}
button {
  margin-top: 10px;
}
</style>
