<template>
  <!-- ToDo List -->
  <div id="todoListPage" class="bg-half">
    <nav>
      <h1><a href="#">ONLINE TODO LIST</a></h1>
      <ul>
        <li class="todo_sm">
          <a href="#"
            ><span>{{ signinData.nickname }}的代辦事項</span></a
          >
        </li>
        <li><a href="#loginPage" @click.prevent="signOut">登出</a></li>
      </ul>
    </nav>
    <div class="conatiner todoListPage vhContainer">
      <div class="todoList_Content">
        <div class="inputBox">
          <input type="text" placeholder="請輸入待辦事項" v-model="newTodo" />
          <a href="#" @click.prevent="CreateTodoList">
            <i class="fa fa-plus"></i>
          </a>
        </div>
        <div class="todoList_list">
          <ul class="todoList_tab">
            <li>
              <a
                href="#"
                :class="activeType === '1' ? 'active' : ''"
                @click.prevent="changeType('1')"
                >全部</a
              >
            </li>
            <li>
              <a
                href="#"
                :class="activeType === '2' ? 'active' : ''"
                @click.prevent="changeType('2')"
                >待完成</a
              >
            </li>
            <li>
              <a
                href="#"
                :class="activeType === '3' ? 'active' : ''"
                @click.prevent="changeType('3')"
                >已完成</a
              >
            </li>
          </ul>
          <!-- 全部項目 -->
          <div class="todoList_items" v-show="activeType === '1'">
            <ul class="todoList_item">
              <li v-for="todo in type1" :key="todo.id">
                <label class="todoList_label">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    value="true"
                    :checked="todo.status"
                    @change="updateStatus(todo.id)"
                  />
                  <span>{{ todo.content }}</span>
                </label>
                <a href="#" @click.prevent="deleteTodo(todo.id)">
                  <i class="fa fa-times"></i>
                </a>
              </li>
            </ul>
            <div class="todoList_statistics">
              <p>{{ type2Count }} 個待完成項目</p>
            </div>
          </div>
          <!-- 待完成 -->
          <div class="todoList_items" v-show="activeType === '2'">
            <ul class="todoList_item">
              <li v-for="todo in type2" :key="todo.id">
                <label class="todoList_label">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    value="true"
                    :checked="todo.status"
                    @change="updateStatus(todo.id)"
                  />
                  <span>{{ todo.content }}</span>
                </label>
                <a href="#" @click.prevent="deleteTodo(todo.id)">
                  <i class="fa fa-times"></i>
                </a>
              </li>
            </ul>
            <div class="todoList_statistics">
              <p>{{ type2Count }} 個待完成項目</p>
            </div>
          </div>
          <!-- 已完成 -->
          <div class="todoList_items" v-show="activeType === '3'">
            <ul class="todoList_item">
              <li v-for="todo in type3" :key="todo.id">
                <label class="todoList_label">
                  <input
                    class="todoList_input"
                    type="checkbox"
                    value="true"
                    :checked="todo.status"
                    @change="updateStatus(todo.id)"
                  />
                  <span>{{ todo.content }}</span>
                </label>
                <a href="#" @click.prevent="deleteTodo(todo.id)">
                  <i class="fa fa-times"></i>
                </a>
              </li>
            </ul>
            <div class="todoList_statistics">
              <p>{{ type3Count }} 個已完成項目</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'
import axios from 'axios'
import { computed, onMounted, ref } from 'vue'
const api = 'https://todolist-api.hexschool.io'
const token = ref('')
const router = useRouter()
const signinData = ref({
  status: '',
  exp: '',
  token: '',
  nickname: ''
})

//   網頁開啟時驗證token，若沒有，則將token清空並回到登入頁面。
onMounted(async () => {
  const cookieToken = document.cookie.replace(
    /(?:(?:^|.*;\s*)customTodoName\s*\=\s*([^;]*).*$)|^.*$/,
    '$1'
  )

  const res = await axios
    .get(`${api}/users/checkout`, {
      headers: {
        Authorization: cookieToken
      }
    })
    .then((res) => {
      signinData.value = res.data
      token.value = cookieToken
      getTodoList()
    })
    .catch(function (err) {
      token.value = ''
      alert('請重新登入：' + err.message)
      setTimeout(() => {
        router.push('/login')
      }, 1500)
    })
})

//   登出
const signOut = async () => {
  try {
    const res = await axios.post(
      `${api}/users/sign_out`,
      {},
      {
        headers: {
          Authorization: token.value
        }
      }
    )
    signinData.value = {}
    token.value = ''
    alert('登出成功！')
    setTimeout(() => {
      router.push('/login')
    }, 1000)
  } catch (err) {
    alert('登出失敗：' + err.message)
  }
}

const activeType = ref('1')
const changeType = (type) => {
  activeType.value = type
}

//   TODO LIST
const todoListObj = ref([])
const newTodo = ref('')

//   取得ToDoList
const getTodoList = async () => {
  try {
    const res = await axios.get(`${api}/todos/`, {
      headers: {
        Authorization: token.value
      }
    })
    todoListObj.value = res.data.data
  } catch (err) {
    alert('資料取得失敗：' + err.message)
  }
}
// 創建項目
const CreateTodoList = async () => {
  const todo = {
    content: newTodo.value
  }
  try {
    const res = await axios.post(`${api}/todos/`, todo, {
      headers: {
        Authorization: token.value
      }
    })
    alert('新增成功！')
    newTodo.value = ''
    getTodoList()
  } catch (err) {
    alert('新增失敗：' + err.message)
  }
}

// 切換欄位
const type1 = computed(() => {
  return todoListObj.value
})
const type1Count = computed(() => {
  return todoListObj.value.length
})
const type2 = computed(() => {
  return todoListObj.value.filter((item) => item.status === false)
})
const type2Count = computed(() => {
  return todoListObj.value.filter((item) => item.status === false).length
})
const type3 = computed(() => {
  return todoListObj.value.filter((item) => item.status === true)
})
const type3Count = computed(() => {
  return todoListObj.value.filter((item) => item.status === true).length
})

// 更新狀態
const updateStatus = async (id) => {
  try {
    // 發送 PATCH 或 PUT 請求來更新狀態
    const res = await axios.patch(
      `${api}/todos/${id}/toggle`,
      {},
      {
        headers: {
          Authorization: token.value
        }
      }
    )
    // 更新本地狀態
    getTodoList();
  } catch (err) {
    alert('狀態更新失敗：' + err.message)
  }
}

// 刪除項目
const deleteTodo = async (index) => {
  try {
    await axios.delete(`${api}/todos/${index}`, {
      headers: {
        Authorization: token.value
      }
    })
    todoListObj.value.splice(index, 1)
    alert('刪除成功！')
  } catch (err) {
    alert('刪除失敗：' + err.message)
  }
}
</script>
