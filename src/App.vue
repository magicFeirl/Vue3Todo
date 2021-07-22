<template>
  <div class="container">
    <!-- 左侧 tabbars -->
    <div class="tabbar-container">
      <div class="tabbar-list">
        <div
          class="tabbar-item"
          :class="{ 'tabbar-active': currentTabIdx === parseInt(key) }"
          v-for="(item, key) in tabs"
          :key="key"
          @click="onSwitchTab(parseInt(key))"
        >
          <div class="tabbar-item-content">
            <span><i class="fa fa-list-ul"></i>{{ item.name }}</span
            ><span style="color: #aaa"> {{ item.todoList.length || "" }}</span>
          </div>
        </div>
      </div>
      <div class="add-tabbar-item" @click="addTabbar('未命名')">
        <i class="fa fa-plus"></i>新建分类
      </div>
    </div>

    <!-- 右侧todolist -->
    <div class="todolist-container">
      <h3 class="title">
        {{ todoTitle }}<i class="fa fa-plus" @click="addTodoItem"></i>
      </h3>
      <div class="todo-list">
        <div
          class="todo-item"
          :class="{ finished: todo.finished }"
          v-for="(todo, idx) in computedTodoList"
          :key="idx"
        >
          <input type="checkbox" v-model="todo.finished" :key="idx" />
          <div>
            <span v-if="!isEditingCurrentTodo(idx)" class="content">{{
              todo.content
            }}</span>
            <input
              placeholder="what's up?"
              autofocus
              class="edit-todo"
              v-model="todo.content"
              @blur="cancelEdit(idx)"
              @keyup.enter="cancelEdit(idx)"
              v-else
            />
          </div>
          <i class="fa fa-edit" @click="editTodo(idx)"></i>
          <i class="fa fa-trash" @click="deleteTodo(idx)"></i>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";

interface todoItem {
  content: string;
  finished: boolean;
  ceatedAt?: string;
  tabId: number;
  id?: number;
}

type todoList = todoItem[];
interface todoTabItem {
  name: string;
  todoList: todoList;
}

interface todoTabList {
  [key: number]: todoTabItem;
}

const tabs: todoTabList = reactive({
  1: {
    name: "待办",
    todoList: [
      {
        content: "组件化 todo list / 代码优化",
        tabId: 1,
        finished: false,
      },
      {
        content: "添加 todo 状态 filter",
        tabId: 1,
        finished: false,
      },
    ],
  },
  2: {
    name: "进行中",
    todoList: [
      {
        content: "breath",
        tabId: 2,
        finished: false,
      },
    ],
  },
});

import { ref, computed, reactive } from "vue";

export default defineComponent({
  name: "App",
  setup() {
    const currentTabIdx = ref(1);
    const todoTitle = ref(tabs[currentTabIdx.value].name);

    const currentTab = computed(() => {
      return tabs[currentTabIdx.value];
    });

    const isEditing = ref(false);
    const currentEditingTodoIdx = ref(0);

    const editTodo = (idx: number) => {
      isEditing.value = true;
      currentEditingTodoIdx.value = idx;
    };

    const isEditingCurrentTodo = (idx: number) => {
      // console.log(isEditing.value, currentEditingTodoIdx.value, idx);
      return isEditing.value && currentEditingTodoIdx.value === idx;
    };

    const deleteTodo = (idx: number) => {
      computedTodoList.value.splice(idx, 1);
    };

    // 当前 todo list
    const computedTodoList = computed(() => {
      return currentTab.value.todoList;
    });

    const onSwitchTab = (tabId: number) => {
      currentTabIdx.value = tabId;
      todoTitle.value = currentTab.value.name;
      // 切换 tab 的时候关闭编辑标志
      isEditing.value = false;
    };

    const toggleTodo = (idx: number) => {
      computedTodoList.value[idx].finished =
        !computedTodoList.value[idx].finished;
    };

    const addTabbar = (name: string) => {
      const id = Object.keys(tabs).length + 1;
      tabs[id + 1] = {
        name,
        todoList: [],
      };
    };

    const cancelEdit = (idx: number) => {
      isEditing.value = false;
      // 如果没有内容
      if (!computedTodoList.value[idx].content.trim()) {
        deleteTodo(idx);
      }
    };

    const addTodoItem = () => {
      // 添加自动编辑todo
      const lastTodoIdx = computedTodoList.value.length;
      if (
        lastTodoIdx === 0 ||
        computedTodoList.value[lastTodoIdx - 1].content
      ) {
        editTodo(lastTodoIdx);
        computedTodoList.value.push(
          reactive({
            content: "",
            finished: false,
            tabId: currentTabIdx.value,
          })
        );
      }
    };

    return {
      currentTabIdx,
      todoTitle,
      computedTodoList,
      tabs,
      isEditing,
      currentEditingTodoIdx,
      cancelEdit,
      onSwitchTab,
      addTodoItem,
      toggleTodo,
      addTabbar,
      editTodo,
      deleteTodo,
      isEditingCurrentTodo,
    };
  },
});
</script>

<style lang="less">
.tabbar-container,
.todolist-container {
  max-height: 100%;
  overflow: auto;
}

.tabbar-container {
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  .add-tabbar-item,
  .tabbar-item {
    padding: 10px;
  }

  .add-tabbar-item {
    display: flex;
    align-items: center;
  }

  .tabbar-item,
  .add-tabbar-item {
    cursor: pointer;
  }

  .fa-list-ul,
  .fa-plus {
    margin-right: 10px;
    font-size: 12px;
  }
}

.tabbar-item-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.todolist-container {
  @fontSize: 16px;
  margin-left: 10px;

  .finished .content {
    text-decoration: line-through;
    color: #888;
  }

  .todo-item {
    font-size: @fontSize;
    padding: 15px 10px;
    background: white;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.4);
    border-radius: 5px;
    display: grid;
    grid-template-columns: 1fr 17fr 1fr 1fr;
    align-items: center;

    .edit-todo {
      width: 100%;
      outline: none;
      border: none;
      font-size: 16px;
    }

    &:hover i {
      display: inline;
      color: #888;
    }

    i:hover {
      color: #444;
    }

    i {
      display: none;
    }

    cursor: pointer;

    &:not(last-child) {
      margin-bottom: 10px;
    }

    input {
      margin-right: 10px;
      height: @fontSize;
      width: @fontSize;
    }
  }

  .title {
    margin-top: 10px;
    display: flex;
    justify-content: space-between;
    padding: 0 5px;
  }
}

.tabbar-list {
  .tabbar-active {
    background-color: #dee4f2;
  }
}

.tabbar-container,
.todolist-container {
  background-color: rgb(243, 244, 246);
  font-size: 14px;
  padding: 20px;
}

.container {
  height: 602px;
  width: 802px;
  display: grid;
  grid-template-columns: 2fr 5fr;
  background-color: white;
}
</style>
