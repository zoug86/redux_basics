<template>
  <div>
    <AddTask v-show="showAddTask" @add-task="addTask" />
    <!-- <AddTask v-if="showAddTask" @add-task="addTask" /> -->
    <Tasks
      @toggle-reminder="toggleReminder"
      @delete-task="deleteTask"
      :tasks="tasks"
    />
  </div>
</template>

<script>
import Tasks from "../components/Tasks";
import AddTask from "../components/AddTask";

export default {
  name: "Home",
  components: {
    Tasks,
    AddTask,
  },
  props: {
    showAddTask: Boolean,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      const response = await fetch("/api/tasks", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(task),
      });
      const newTask = await response.json();
      this.tasks.push(newTask);
    },

    async deleteTask(id) {
      const response = await fetch(`/api/tasks/${id}`, {
        method: "DELETE",
      });
      // const deletedTask = await response.json();
      // console.log(deletedTask);
      this.tasks = this.tasks.filter((task) => task.id !== id);
    },

    async toggleReminder(id) {
      // 1st Method
      // const response = await fetch(`/api/tasks/${id}`, {
      //   method: "PATCH",
      //   headers: {
      //     "Content-Type": "application/json",
      //   },
      //   body: JSON.stringify({
      //     reminder: !this.tasks.find((task) => task.id === id).reminder,
      //   }),
      // });
      // 2nd Method
      const taskToToggle = await this.fetchTask(id);
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder };
      const response = await fetch(`/api/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(updTask),
      });
      const updatedTask = await response.json();
      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: updatedTask.reminder } : task
      );
    },
    async fetchTasks() {
      const response = await fetch("api/tasks");
      const data = await response.json();
      return data;
    },
    async fetchTask(id) {
      const response = await fetch(`api/tasks/${id}`);
      const data = await response.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>