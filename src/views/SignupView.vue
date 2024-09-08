<template>
  <!-- sign up -->
  <div id="signUpPage" class="bg-yellow">
    <div class="conatiner signUpPage vhContainer">
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
          <h2 class="formControls_txt">註冊帳號</h2>
          <label class="formControls_label" for="email">Email</label>
          <input
            class="formControls_input"
            type="email"
            id="email"
            name="email"
            placeholder="請輸入 email"
            v-model="signupField.email"
            required
          />
          <label class="formControls_label" for="name">您的暱稱</label>
          <input
            class="formControls_input"
            type="text"
            name="name"
            id="name"
            placeholder="請輸入您的暱稱"
            v-model="signupField.nickname"
          />
          <label class="formControls_label" for="pwd">密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="pwd"
            placeholder="請輸入密碼"
            v-model="signupField.password"
            required
          />
          <label class="formControls_label" for="pwd">再次輸入密碼</label>
          <input
            class="formControls_input"
            type="password"
            name="pwd"
            id="pwd"
            placeholder="請再次輸入密碼"
            v-model="signupField.confirmPassword"
            required
          />
          <input class="formControls_btnSubmit" type="button" value="註冊帳號" @click="signup" />
          <!-- <a class="formControls_btnLink" href="#loginPage">登入</a> -->
          <RouterLink class="formControls_btnLink" to="/login">登入</RouterLink>
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
const router = useRouter()

//   註冊
const signupField = ref({
  email: '',
  password: '',
  confirmPassword: '',
  nickname: ''
})

const emailRule = /^\w+((-\w+)|(\.\w+))*\@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z]+$/;

const signup = async () => {
  if (signupField.value.password !== signupField.value.confirmPassword) {
    alert('輸入的密碼不一致，請重新確認')
    return
  }else if (!emailRule.test(signupField.value.email)) {
    alert('請輸入正確的信箱格式')
    return
  }

  try {
    const res = await axios.post(`${api}/users/sign_up`, signupField.value)
    alert('註冊成功，請重新登入！')

    setTimeout(() => {
      router.push('/login')
    }, 1000)
  } catch (err) {
    alert('註冊失敗：' + err.message)
  }
}
</script>
