<template>
  <div class="container">
    <!-- 左侧 tabbars -->
    <div class="tabbar-container">
      <div class="tabbar-list">
        <div
          class="tabbar-item"
          :class="{ 'tabbar-active': isActiveTabbar(parseInt(key)) }"
          v-for="(item, key) in tabs"
          :key="key"
        >
          <div class="tabbar-item-content" v-if="!!item">
            <div
              style="
                display: grid;
                grid-template-columns: 1fr 9fr;
                align-items: center;
              "
              @click="onSwitchTab(parseInt(key))"
            >
              <i class="fa fa-list-ul" style="margin-top: 2px"></i
              ><editable-content
                class="edit-todo"
                :edit="isEditingTab && isActiveTabbar(parseInt(key))"
                v-model="item.name"
                @blur="cancelEditTab"
                @keyup.enter="cancelEditTab"
                ><span @dblclick.prevent="editTabbarItem">{{
                  item.name
                }}</span></editable-content
              >
            </div>
            <div class="tab-opt">
              <span class="todo-count"> {{ item.todoList.length }} </span>
              <span
                class="delete-tab"
                @click.prevent="deleteTab(parseInt(key))"
              >
                <i class="fa fa-trash"></i>
              </span>
            </div>
          </div>
        </div>
      </div>
      <div class="add-tabbar-item" @click="addTabbar('双击重命名')">
        <i class="fa fa-plus"></i>新建分类
      </div>
    </div>
    <!-- 右侧todolist -->
    <div class="todolist-container">
      <h3 class="title">
        {{ todoTitle }}
        <div class="todo-opt">
          <span
            :class="{ 'active-filter': filterType === 'all' }"
            @click="filterTodo('all')"
            >全部</span
          >
          <span
            :class="{ 'active-filter': filterType === 'undo' }"
            @click="filterTodo('undo')"
            >未完成</span
          >
          <span
            :class="{ 'active-filter': filterType === 'done' }"
            @click="filterTodo('done')"
            >已完成</span
          >
          <i
            class="fa fa-plus"
            :class="{ disabled: filterType !== 'all' }"
            @click="addTodoItem"
          />
        </div>
      </h3>
      <div class="todo-list">
        <div
          class="todo-item"
          :class="{ finished: todo.finished }"
          v-for="(todo, idx) in computedTodoList"
          :key="idx"
        >
          <input
            type="checkbox"
            v-model="todo.finished"
            :key="todo.content + idx"
          />
          <div>
            <editable-content
              class="edit-todo"
              autofoucs
              v-model="todo.content"
              @blur="cancelEdit(idx)"
              @keyup.enter="cancelEdit(idx)"
              :edit="isEditingCurrentTodo(idx)"
            >
              <span class="content" @click="editTodo(idx)">{{
                todo.content
              }}</span>
            </editable-content>
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

let tabs: todoTabList = reactive({
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
      {
        content: "fix option header",
        tabId: 1,
        finished: false,
      },
    ],
  },
  3: {
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

type todoFilter = "all" | "done" | "undo";

import { ref, computed, reactive, watch } from "vue";
import EditableContent from "@/components/EditableContent.vue";

export default defineComponent({
  name: "App",
  components: {
    EditableContent,
  },
  setup() {
    const currentTabIdx = ref(1);
    const isEditingTab = ref(false);
    const todoTitle = ref(tabs[currentTabIdx.value].name);

    const isActiveTabbar = (idx: number) => {
      return currentTabIdx.value === idx;
    };

    const deleteTab = (key: number) => {
      currentTabIdx.value = 0;

      delete tabs[key];
    };

    const editTabbarItem = () => {
      isEditingTab.value = true;
    };

    const cancelEditTab = () => {
      isEditingTab.value = false;
    };

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

    const filterType = ref<todoFilter>("all");
    const filterTodo = (type: todoFilter) => {
      filterType.value = type;
    };

    // 当前 todo list
    const computedTodoList = computed(() => {
      if (!currentTab.value) {
        return [];
      }

      if (filterType.value === "all") {
        return currentTab.value.todoList;
      } else if (filterType.value === "done") {
        return currentTab.value.todoList.filter((x) => x.finished);
      } else {
        return currentTab.value.todoList.filter((x) => !x.finished);
      }
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
      let id;
      const keys = Object.keys(tabs).sort();

      if (!keys.length) {
        id = 1;
      } else {
        id = Number(keys[keys.length - 1]) + 1;
      }

      tabs[id] = {
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
      // 添加自动编辑todo，限定只能在 all tab 下添加 todo
      if (filterType.value !== "all") {
        return;
      }
      const lastTodoIdx = computedTodoList.value.length;
      if (
        lastTodoIdx === 0 ||
        computedTodoList.value[lastTodoIdx - 1].content
      ) {
        editTodo(lastTodoIdx);
        computedTodoList.value.push({
          content: "",
          finished: false,
          tabId: currentTabIdx.value,
        });
      }
    };

    return {
      currentTabIdx,
      todoTitle,
      computedTodoList,
      tabs,
      isEditing,
      currentEditingTodoIdx,
      isEditingTab,
      filterType,
      filterTodo,
      deleteTab,
      cancelEditTab,
      editTabbarItem,
      isActiveTabbar,
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
.disabled {
  color: #ccc !important;
  cursor: default !important;
}

.todo-opt {
  span,
  i {
    margin-right: 10px;
    display: inline-block;
    cursor: pointer;
  }

  span {
    color: #aaa;
    font-size: 14px;
  }

  .active-filter {
    color: #666;
  }
}

.tabbar-container {
  display: flex;
  flex-direction: column;
  justify-content: space-between;

  .tab-opt {
    color: #aaa;
    .delete-tab {
      display: none;
    }

    &:hover .delete-tab {
      display: inline;
    }

    &:hover .todo-count {
      display: none;
    }
  }

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

.edit-todo {
  width: 100% !important;
  outline: none;
  border: none;
  font-size: inherit;
  background: inherit;
  color: #444;
  font-style: italic;
}

.todolist-container {
  @fontSize: 16px;
  @titleHeight: 20px;
  margin-left: 10px;

  .finished .content {
    text-decoration: line-through;
    color: #888;
  }

  .todo-item {
    font-size: @fontSize;
    padding: 15px 10px;
    background: white;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
    border-radius: 5px;
    display: grid;
    grid-template-columns: 1fr 17fr 1fr 1fr;
    align-items: center;

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
    height: @titleHeight;
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
  overflow: auto;
  max-height: 100%;
}

.container {
  height: 602px;
  width: 802px;
  display: grid;
  grid-template-columns: 2fr 5fr;
  background-color: white;
}
</style>
