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
        <div class="todoList_list" v-if="todoListObj.length > 0">
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
              <li v-for="todo in allItem" :key="todo.id">
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
              <p>{{ notOverCount }} 個待完成項目</p>
            </div>
          </div>
          <!-- 待完成 -->
          <div class="todoList_items" v-show="activeType === '2'">
            <ul class="todoList_item">
              <li v-for="todo in notOver" :key="todo.id">
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
              <p>{{ notOverCount }} 個待完成項目</p>
            </div>
          </div>
          <!-- 已完成 -->
          <div class="todoList_items" v-show="activeType === '3'">
            <ul class="todoList_item">
              <li v-for="todo in Over" :key="todo.id">
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
              <p>{{ OverCount }} 個已完成項目</p>
            </div>
          </div>
        </div>
        <div class="noitem" v-else>目前尚無待辦事項</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'
import axios from 'axios'
import Swal from 'sweetalert2'
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
      Swal.fire({
        icon: 'error',
        title: '請重新登入',
        showConfirmButton: false,
        timer: 1500
      })
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
    Swal.fire({
      title: '期待下次再見',
      icon: 'success',
      showConfirmButton: false,
      timer: 1500
    })
    signinData.value = {}
    token.value = ''
    setTimeout(() => {
      router.push('/login')
    }, 1500)
  } catch (err) {
    Swal.fire({
      icon: 'error',
      title: '登出失敗'
    })
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
    Swal.fire({
      icon: 'error',
      title: '資料取得失敗'
    })
  }
}
// 創建項目
const CreateTodoList = async () => {
  const todo = {
    content: newTodo.value
  }

  // 先檢查內容是否為空白
  if (!todo.content.trim()) {
    Swal.fire({
      icon: 'error',
      title: '內容不可以是空白唷'
    })
    return
  }

  try {
    const res = await axios.post(`${api}/todos/`, todo, {
      headers: {
        Authorization: token.value
      }
    })
    Swal.fire({
      title: '新增成功！',
      icon: 'success',
      showConfirmButton: false,
      timer: 1000
    })
    newTodo.value = ''
    getTodoList()
  } catch (err) {
    Swal.fire({
      icon: 'error',
      title: '資料新增失敗！'
    })
  }
}

// 切換欄位
const allItem = computed(() => {
  return todoListObj.value
})
const notOver = computed(() => {
  return todoListObj.value.filter((item) => item.status === false)
})
const notOverCount = computed(() => {
  return todoListObj.value.filter((item) => item.status === false).length
})
const Over = computed(() => {
  return todoListObj.value.filter((item) => item.status === true)
})
const OverCount = computed(() => {
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
    getTodoList()
  } catch (err) {
    Swal.fire({
      icon: 'error',
      title: '狀態更新失敗！'
    })
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
    Swal.fire({
      title: '刪除成功！',
      icon: 'success',
      showConfirmButton: false,
      timer: 1000
    })
  } catch (err) {
    Swal.fire({
      icon: 'error',
      title: '刪除失敗！'
    })
  }
}
</script>
