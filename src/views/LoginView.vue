<template>
  <!-- login_page -->
  <div id="loginPage" class="bg-yellow">
    <div class="conatiner loginPage vhContainer">
      <div class="side">
        <a href="#"
          ><img
            class="logoImg"
            src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png"
            alt=""
        /></a>
        <img
          class="d-m-n"
          src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png"
          alt="workImg"
        />
      </div>
      <div>
        <form class="formControls" action="index.html">
          <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>
          <label class="formControls_label" for="email">Email</label>
          <input
            class="formControls_input"
            type="text"
            id="email"
            name="email"
            placeholder="請輸入 email"
            v-model="signInField.email"
            required
          />
          <span>此欄位不可留空</span>
          <label class="formControls_label" for="pwd">密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="pwd"
            placeholder="請輸入密碼"
            v-model="signInField.password"
            required
          />
          <input class="formControls_btnSubmit" type="button" value="登入" @click="signin" />
          <!-- <a class="formControls_btnLink" @click="signShow">註冊帳號</a> -->
          <RouterLink class="formControls_btnLink" to="/signup">註冊帳號</RouterLink>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { RouterLink, useRouter } from 'vue-router'
import axios from 'axios'
import { ref } from 'vue'
const api = 'https://todolist-api.hexschool.io'
const token = ref('')
const router = useRouter()

//   登入
const signInField = ref({
  email: '',
  password: ''
})

const signinData = ref({
  status: '',
  exp: '',
  token: '',
  nickname: ''
})

const signin = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_in`, signInField.value)
    signinData.value = res.data
    token.value = res.data.token
    document.cookie = `customTodoName=${res.data.token}`
    alert(`登入成功～${signinData.value.nickname}歡迎回來！`)
    signInField.value = {}
    setTimeout(() => {
      router.push('/todolist')
    }, 1000)
  } catch (err) {
    alert('登入失敗：' + err.message)
  }
}
</script>
