<template>
  <div class="app">
    <h1>我的Todo List</h1>

    <!-- 添加任务 -->
    <div class="add-task">
      <input 
        v-model="taskText" 
        placeholder="输入任务..." 
        @keyup.enter="add" 
        class="input-text"
      />
      <select v-model="taskType" class="select-type">
        <option value="工作">工作</option>
        <option value="生活">生活</option>
        <option value="学习">学习</option>
      </select>
      <input 
        v-model="taskTime" 
        type="date" 
        class="input-date"
        :min="nowDate"
      />
      <button @click="add">添加</button>
    </div>

    <!-- 搜索 -->
    <div class="search-box">
      <input 
        v-model="searchText" 
        placeholder="搜索..." 
        class="input-search"
      />
      <select v-model="selectType" class="select-filter">
        <option value="">全部分类</option>
        <option value="工作">工作</option>
        <option value="生活">生活</option>
        <option value="学习">学习</option>
      </select>
      <select v-model="selectStatus" class="select-filter">
        <option value="">全部状态</option>
        <option value="pending">未完成</option>
        <option value="completed">已完成</option>
      </select>
    </div>

    <!-- 统计 -->
    <div class="count-info">
      <span>总共: {{ allTasks.length }}</span>
      <span>完成: {{ doneCount }}</span>
      <span>未完成: {{ todoCount }}</span>
    </div>

    <!-- 任务列表 -->
    <ul class="task-items">
      <li 
        v-for="item in showTasks" 
        :key="item.id" 
        :class="{ 
          done: item.isDone,
          late: checkLate(item.time) && !item.isDone
        }"
      >
        <div class="task-info">
          <input 
            type="checkbox" 
            v-model="item.isDone" 
            @change="save" 
          />
          <span class="text">{{ item.text }}</span>
          <span class="type" :class="getTypeClass(item.type)">
            {{ item.type }}
          </span>
          <span v-if="item.time" class="time">
            📅 {{ showDate(item.time) }}
            <span v-if="checkLate(item.time) && !item.isDone" class="late-text">
              过期了
            </span>
          </span>
        </div>
        <button @click="remove(item.id)" class="btn-delete">删除</button>
      </li>
    </ul>

    <div v-if="showTasks.length === 0" class="no-task">
      {{ getNoTaskText() }}
    </div>

    <!-- 按钮 -->
    <div class="buttons">
      <button @click="removeAll" class="btn-clear">清空全部</button>
      <button @click="removeLate" class="btn-late">删除过期</button>
      <button @click="removeDone" class="btn-done">删除完成</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      taskText: '',
      taskType: '工作',
      taskTime: '',
      allTasks: [],
      searchText: '',
      selectType: '',
      selectStatus: '',
    };
  },
  computed: {
    nowDate() {
      return new Date().toISOString().split('T')[0];
    },
    showTasks() {
      let result = this.allTasks;

      if (this.searchText.trim()) {
        result = result.filter(item => 
          item.text.toLowerCase().includes(this.searchText.toLowerCase())
        );
      }

      if (this.selectType) {
        result = result.filter(item => item.type === this.selectType);
      }

      if (this.selectStatus === 'completed') {
        result = result.filter(item => item.isDone);
      } else if (this.selectStatus === 'pending') {
        result = result.filter(item => !item.isDone);
      }

      return result;
    },
    doneCount() {
      return this.allTasks.filter(item => item.isDone).length;
    },
    todoCount() {
      return this.allTasks.filter(item => !item.isDone).length;
    }
  },
  mounted() {
    this.load();
    this.checkTime();
    setInterval(this.checkTime, 60000);
  },
  methods: {
    add() {
      if (this.taskText.trim()) {
        const newItem = {
          id: Date.now(),
          text: this.taskText.trim(),
          type: this.taskType,
          time: this.taskTime || null,
          isDone: false,
          createTime: new Date().toISOString()
        };
        
        this.allTasks.push(newItem);
        this.save();
        this.taskText = '';
        this.taskTime = '';
      }
    },

    remove(id) {
      this.allTasks = this.allTasks.filter(item => item.id !== id);
      this.save();
    },

    removeAll() {
      if (confirm('确定删除所有任务吗？')) {
        this.allTasks = [];
        this.save();
      }
    },

    removeLate() {
      const lateTasks = this.allTasks.filter(item => 
        this.checkLate(item.time) && !item.isDone
      );
      
      if (lateTasks.length === 0) {
        alert('没有过期任务');
        return;
      }

      if (confirm(`确定删除 ${lateTasks.length} 个过期任务吗？`)) {
        this.allTasks = this.allTasks.filter(item => 
          !this.checkLate(item.time) || item.isDone
        );
        this.save();
      }
    },

    removeDone() {
      const doneTasks = this.allTasks.filter(item => item.isDone);
      
      if (doneTasks.length === 0) {
        alert('没有完成的任务');
        return;
      }

      if (confirm(`确定删除 ${doneTasks.length} 个完成任务吗？`)) {
        this.allTasks = this.allTasks.filter(item => !item.isDone);
        this.save();
      }
    },

    checkLate(time) {
      if (!time) return false;
      return new Date(time) < new Date().setHours(0, 0, 0, 0);
    },

    showDate(dateStr) {
      if (!dateStr) return '';
      const date = new Date(dateStr);
      return date.toLocaleDateString('zh-CN');
    },

    getTypeClass(type) {
      const map = {
        '工作': 'type-work',
        '生活': 'type-life',
        '学习': 'type-study'
      };
      return map[type] || '';
    },

    getNoTaskText() {
      if (this.searchText) {
        return `没找到"${this.searchText}"相关任务`;
      }
      if (this.selectType || this.selectStatus) {
        return '没有符合条件的任务';
      }
      return '还没有任务，快添加一个吧！';
    },

    save() {
      localStorage.setItem('myTasks', JSON.stringify(this.allTasks));
    },

    load() {
      const data = localStorage.getItem('myTasks');
      if (data) {
        this.allTasks = JSON.parse(data);
      }
    },
  },
};
</script>

<style scoped>
.app {
  max-width: 800px;
  margin: 20px auto;
  padding: 30px;
  background: #f8f9fa;
  border-radius: 12px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.app h1 {
  text-align: center;
  color: #2c3e50;
  font-size: 2rem;
  margin-bottom: 30px;
  font-weight: 600;
}

.add-task, .search-box {
  display: flex;
  gap: 12px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.input-text {
  flex: 2;
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 6px;
  min-width: 200px;
}

.input-text:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.1);
}

.select-type, .input-date, .input-search, .select-filter {
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  background: white;
}

.select-type:focus, .input-date:focus, .input-search:focus, .select-filter:focus {
  outline: none;
  border-color: #007bff;
}

.input-search {
  flex: 1;
  min-width: 150px;
}

.add-task button {
  padding: 12px 20px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
}

.add-task button:hover {
  background: #0056b3;
}

.count-info {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.count-info span {
  background: white;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  color: #666;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.task-items {
  list-style: none;
  padding: 0;
  margin-bottom: 20px;
}

.task-items li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
  border-radius: 8px;
  margin-bottom: 12px;
  background: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease;
}

.task-items li:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.task-items li.done {
  background: #f8f9fa;
  opacity: 0.8;
}

.task-items li.late {
  border-left: 4px solid #dc3545;
  background: #fff5f5;
}

.task-info {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
}

.task-info input[type="checkbox"] {
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.text {
  font-size: 16px;
  color: #333;
}

.task-items li.done .text {
  text-decoration: line-through;
  color: #999;
}

.type {
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  color: white;
  font-weight: 500;
}

.type-work { background: #007bff; }
.type-life { background: #28a745; }
.type-study { background: #ffc107; color: #333; }

.time {
  font-size: 13px;
  color: #666;
}

.late-text {
  color: #dc3545;
  font-weight: 600;
  font-size: 11px;
}

.btn-delete {
  padding: 6px 12px;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.btn-delete:hover {
  background: #c82333;
}

.no-task {
  text-align: center;
  color: #999;
  padding: 40px;
  font-style: italic;
  background: white;
  border-radius: 8px;
}

.buttons {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  justify-content: center;
}

.buttons button {
  padding: 10px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  color: white;
}

.btn-clear { background: #6c757d; }
.btn-late { background: #dc3545; }
.btn-done { background: #28a745; }

.buttons button:hover {
  opacity: 0.9;
}

@media (max-width: 600px) {
  .app {
    margin: 10px;
    padding: 20px;
  }
  
  .add-task, .search-box {
    flex-direction: column;
  }
  
  .count-info {
    flex-direction: column;
    align-items: center;
    gap: 8px;
  }
  
  .task-info {
    flex-wrap: wrap;
    gap: 8px;
  }
  
  .buttons {
    flex-direction: column;
  }
}
</style>
