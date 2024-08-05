<template>
  <div class="calendar">
    <div
        v-for="(day, index) in days"
        :key="index"
        class="day"
        @drop="onDrop(day)"
        @dragover.prevent
    >
      <div class="date">{{ day.format('YYYY-MM-DD') }}</div>
      <div class="tasks">
        <div
            v-for="task in tasks.filter(t => t.date === day.format('YYYY-MM-DD'))"
            :key="task.id"
            class="task"
            draggable="true"
            @dragstart="onDragStart(task)"
        >
          {{ task.name }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import dayjs from 'dayjs';
import { ref } from 'vue';

export default {
  name: 'WeeklyCalendar',
  props: {
    tasks: Array,
    draggedTask: Object
  },
  emits: ['task-dropped'],
  setup(props, { emit }) {
    const days = ref([]);

    // Initialize 7 days
    for (let i = 0; i < 7; i++) {
      days.value.push(dayjs().add(i, 'day'));
    }

    const onDragStart = (task) => {
      // Save the dragged task's ID to DataTransfer object
      const dataTransfer = event.dataTransfer;
      dataTransfer.setData('taskId', task.id);

      // Optional: Add custom data to DataTransfer
      dataTransfer.setData('text/plain', task.name);
    };

    const onDrop = (day) => {
      const taskId = event.dataTransfer.getData('taskId');
      const task = props.tasks.find(t => t.id === taskId);
      if (task) {
        emit('task-dropped', task, day.format('YYYY-MM-DD'));
      }
    };

    return {
      days,
      onDragStart,
      onDrop
    };
  }
};
</script>

<style>
.calendar {
  display: flex;
  flex-wrap: wrap;
}
.day {
  width: 100px;
  height: 100px;
  border: 1px solid #ccc;
  margin: 5px;
  padding: 5px;
}
.date {
  font-weight: bold;
}
.tasks {
  margin-top: 10px;
}
.task {
  background: #f0f0f0;
  padding: 5px;
  margin-bottom: 5px;
  cursor: grab;
}
</style>
