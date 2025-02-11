<template>
  <div class="todo-container">
    <h2>待办事项</h2>
        <input
      v-model="searchQuery"
      placeholder="🔍 搜索任务..."
      class="search-box"
    />
    <div class="input-container">
      <input v-model="newTask" placeholder="输入任务..." @keyup.enter="addTask" />
      <select v-model="priority">
        <option value="high">🔥 高</option>
        <option value="medium">⚡ 中</option>
        <option value="low">✅ 低</option>
      </select>
      <input type="date" v-model="dueDate" />
      <button @click="addTask">添加</button>
    </div>

    <!-- 清空按钮 -->
    <button class="clear-button" @click="clearCompletedTasks" v-if="tasks.some(task => task.completed)">
      清空已完成任务
    </button>
    <!-- 导出和导入任务按钮 -->
    <button class="export-button" @click="exportTasks">导出任务数据</button>
    <input type="file" @change="importTasks" class="import-input" />
    <ul>
<transition-group name="list" tag="ul">
<li v-for="(task, index) in filteredTasks" :key="index" :class="task.priority">
        <span :class="{ completed: task.completed }" @click="toggleTask(index)">
          {{ task.text }} ({{ getPriorityText(task.priority) }})
          <small v-if="task.dueDate">🗓 截止日期: {{ task.dueDate }}</small>
          <small v-if="task.completedTime">✅ 完成时间: {{ task.completedTime }}</small>
        </span>
        <button v-if="editingIndex !== index" @click="editTask(index)">编辑</button>
        <button v-if="editingIndex === index" @click="saveEdit(index)">保存</button>
        <button @click="removeTask(index)">删除</button>
      </li>
     </transition-group>
    </ul>
  </div>
</template>


<script>
export default {
  data() {
    return {
      newTask: '',
      priority: 'medium',
      dueDate: '',
      searchQuery: '',  // 搜索框输入内容
      tasks: JSON.parse(localStorage.getItem('tasks')) || [],
      editingIndex: null,
      editText: ''
    };
  },
  computed: {
    sortedTasks() {
      return [...this.tasks].sort((a, b) => {
        const priorityOrder = { high: 3, medium: 2, low: 1 };
        if (priorityOrder[a.priority] !== priorityOrder[b.priority]) {
          return priorityOrder[b.priority] - priorityOrder[a.priority];
        }
        if (a.dueDate && b.dueDate) {
          return new Date(a.dueDate) - new Date(b.dueDate);
        }
        return 0;
      });
    },
    filteredTasks() {
      return this.sortedTasks.filter(task =>
        task.text.toLowerCase().includes(this.searchQuery.toLowerCase()) // 任务名匹配搜索关键词
      );
    }
  },
  methods: {
      showToast(message) {
    alert(message); // 简单方式：用 alert 提示
  },
    addTask() {
      if (this.newTask.trim()) {
        this.tasks.push({
          text: this.newTask,
          completed: false,
          priority: this.priority,
          dueDate: this.dueDate,
          completedTime: null
        });
        this.newTask = '';
        this.priority = 'medium';
        this.dueDate = '';
        this.saveTasks();
        this.showToast("✅ 任务已添加！");
      }
    },
    toggleTask(index) {
      this.tasks[index].completed = !this.tasks[index].completed;
      if (this.tasks[index].completed) {
        this.tasks[index].completedTime = new Date().toLocaleString();
      } else {
        this.tasks[index].completedTime = null;
      }
      this.saveTasks();
    },
    removeTask(index) {
      this.tasks.splice(index, 1);
      this.saveTasks();
      this.showToast("❌ 任务已删除！");
    },
    clearCompletedTasks() {
      this.tasks = this.tasks.filter(task => !task.completed);
      this.saveTasks();
      this.showToast("✅ 已清空所有已完成任务！");
    },
    editTask(index) {
      this.editingIndex = index;
      this.editText = this.tasks[index].text;
    },
    saveEdit(index) {
      if (this.editText.trim()) {
        this.tasks[index].text = this.editText;
        this.editingIndex = null;
        this.saveTasks();
      }
    },
    saveTasks() {
      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    },
    exportTasks() {
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.tasks, null, 2));
      const downloadAnchor = document.createElement("a");
      downloadAnchor.setAttribute("href", dataStr);
      downloadAnchor.setAttribute("download", "tasks.json");
      document.body.appendChild(downloadAnchor);
      downloadAnchor.click();
      document.body.removeChild(downloadAnchor);
      this.showToast("✅ 备份成功！");
    },
    importTasks(event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const importedTasks = JSON.parse(e.target.result);
          if (Array.isArray(importedTasks)) {
            this.tasks = importedTasks;
            this.saveTasks();
            alert("任务数据导入成功！");
          } else {
            this.showToast("⚠️ 文件格式错误，请选择正确的 JSON 文件！");
          }
        } catch (error) {
          this.showToast("❌ 导入失败，请确保 JSON 格式正确！");
        }
      };
      reader.readAsText(file);
    },
    getPriorityText(priority) {
      if (priority === 'high') return '🔥 高';
      if (priority === 'medium') return '⚡ 中';
      if (priority === 'low') return '✅ 低';
      return '';
    }
  }
};
</script>




<style scoped>
.todo-container {
  max-width: 450px;
  margin: auto;
  text-align: center;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.search-box {
  width: 90%;
  padding: 10px;
  margin-bottom: 10px;
  border: 2px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
}
.input-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 5px;
}

input, select {
  padding: 8px;
  font-size: 16px;
}

button {
  padding: 8px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background: #218838;
}

ul {
  list-style: none;
  padding: 0;
  margin-top: 20px;
}

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: white;
  padding: 10px;
  margin-bottom: 5px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

li:hover {
  background: #f1f1f1;
}

.completed {
  text-decoration: line-through;
  color: gray;
}

/* 高优先级任务（红色） */
.high {
  border-left: 5px solid red;
}

/* 中优先级任务（橙色） */
.medium {
  border-left: 5px solid orange;
}

/* 低优先级任务（绿色） */
.low {
  border-left: 5px solid green;
}

/* 截止日期 & 完成时间样式 */
small {
  display: block;
  font-size: 14px;
  color: #666;
  margin-top: 5px;
}
/* 进入和离开动画 */
.list-enter-active, .list-leave-active {
  transition: all 0.5s ease;
}
.list-enter, .list-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>
