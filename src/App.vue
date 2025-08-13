<template>
  <div class="container vh-100">
    <div class="row justify-content-center align-items-center h-100">
      <!-- 登入/註冊介面 -->
      <div class="col-md-4 mb-5" v-if="!loginStatus" data-aos="flip-left">
        <div class="header-logo flex-shrink-0 mb-2 w-100 my-0">
          <!-- <img class="mb-1" src="assets/Image/fj-logo.png" alt="logo" /> -->
          <h1 class="text-center pt-0 fw-bolder mb-0 fs-2">待辦清單功能</h1>
          <div class="mb-4 text-center fw-bolder fs-5">這週也有寫作業</div>
        </div>
        <form class="">
          <input
            name="LoginForm"
            type="hidden"
            id="RandomSession"
            value="ea08d2b7-2e65-419d-bf86-d701f41402fa"
          />

          <div class="form-floating mb-2">
            <input
              type="text"
              name="LoginForm"
              class="form-control rounded-top"
              id="LoginForm-AC"
              placeholder="請輸入帳號 "
              v-model="account"
            />
            <label for="floatingInput">帳號</label>
          </div>

          <div class="form-floating mb-2">
            <input
              type="password"
              name="LoginForm"
              class="form-control rounded-bottom"
              id="LoginForm-PW"
              placeholder="請輸入密碼"
              autocomplete="off"
              v-model="password"
            />
            <label for="floatingPassword">密碼</label>
          </div>

          <div class="form-floating mb-2" v-if="isRegisted">
            <input
              type="text"
              name="LoginForm"
              class="form-control rounded-bottom"
              id="LoginForm-NM"
              placeholder="請輸入暱稱"
              autocomplete="off"
              v-model="nickname"
            />
            <label for="floatingPassword">暱稱</label>
          </div>
          <button
            id="loginBtn"
            class="w-100 py-1 btn btn-lg btn-primary link-light mb-2"
            name="LoginForm"
            type="button"
            @click="isRegisted ? signUp() : signIn()"
          >
            {{ isRegisted ? '註冊' : '登入' }}
          </button>
          <button
            id="loginBtn"
            class="w-100 py-1 btn btn-lg btn-secondary link-light mb-2"
            name="LoginForm"
            type="button"
            @click="toggleRegistedUI"
          >
            {{ isRegisted ? '返回登入' : '註冊' }}
          </button>

          <!-- 帳號︰{{ signUpData.email }} 密碼︰{{ signUpData.password }} 暱稱︰{{
            signUpData.nickname
          }} -->
        </form>
      </div>

      <!-- 驗證介面 -->
      <div class="col-md-4 mb-5" v-else-if="!isCheckOut" data-aos="fade-left">
        <div class="header-logo flex-shrink-0 mb-2 w-100 my-0" data-aos="fade-left">
          <!-- <img class="mb-1" src="assets/Image/fj-logo.png" alt="logo" /> -->
          <h1 class="text-center pt-0 fw-bolder mb-0 fs-2 mb-4">驗證帳號資訊</h1>
          <form @submit.prevent="handleSubmit">
            <div class="form-floating mb-2">
              <input
                type="text"
                name="LoginForm"
                class="form-control rounded-top"
                id="LoginForm-VD"
                placeholder="請輸入驗證碼"
                v-model="token"
              />
              <label for="floatingInput">驗證碼</label>
            </div>
            <div class="d-grid gap-2">
              <button type="submit" class="btn btn-primary" @click="checkOut()">驗證</button>
              <button type="button" class="btn btn-secondary" @click="backToLogin()">
                返回登入
              </button>
            </div>
          </form>
        </div>
      </div>

      <!-- 待辦介面 -->
      <div class="col-md-6 mb-5" v-else data-aos="fade-left">
        <h1 class="text-center pt-0 fw-bolder mb-0 fs-2 mb-4">我的待辦清單</h1>
        <ul class="list-group">
          <li class="list-group-item d-flex justify-content-between align-items-center">
            <span></span>
            <input type="text" class="me-1" />
            <div>
              <button class="btn btn-primary btn-sm me-1">新增</button>
              <button class="btn btn-warning btn-sm me-1">編輯</button>
              <button class="btn btn-danger btn-sm">刪除</button>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import AOS from 'aos'
import 'aos/dist/aos.css'
import { ref } from 'vue'
import axios from 'axios'

const isRegisted = ref(false) // 介面狀態
const account = ref('')
const password = ref('')
const nickname = ref('')
const loginStatus = ref(false) // 登入狀態
const validStatus = ref(false) // 驗證狀態
const isCheckOut = ref(false)
const token = ref('')

//
// 登入登出功能
// -------------------------------------------------------------
//
// 註冊資料
const signUpData = ref({
  email: '',
  password: '',
  nickname: '',
})
// 登入資料
const signInData = ref({
  email: '',
  password: '',
})
// API
const url = 'https://todolist-api.hexschool.io'

// 1. 點擊註冊 / 返回登入介面轉換
const toggleRegistedUI = () => {
  isRegisted.value = !isRegisted.value
}
// 2. 註冊功能
const signUp = async () => {
  try {
    const res = await axios.post(`${url}/users/sign_up`, {
      email: account.value,
      password: password.value,
      nickname: nickname.value,
    })
    alert('註冊成功')
    toggleRegistedUI()
    account.value = ``
    password.value = ``
    nickname.value = ``
  } catch (error) {
    alert('註冊失敗')
    console.log(error.response.data)
  }
}
// 3. 登入功能
const signIn = async () => {
  try {
    const response = await axios.post(`${url}/users/sign_in`, {
      email: account.value,
      password: password.value,
    })
    token.value = response.data.token
    alert('登入成功')
    account.value = ''
    password.value = ''
    loginStatus.value = true
  } catch (error) {
    alert('登入失敗')
    console.log(error.response.data)
  }
}
// 4. 驗證功能
const checkOut = async () => {
  try {
    if (!token.value) {
      alert('Token is missing')
      return
    }
    const response = await axios.get(`${url}/users/checkout`, {
      headers: {
        Authorization: token.value,
      },
    })
    isCheckOut.value = true
    alert('驗證成功 UID: ' + response.data.uid)
    getTodo()
  } catch (error) {
    alert('驗證失敗: ' + error.message)
  }
}
const backToLogin = () => {
  token.value = ''
  isRegisted.value = false
  loginStatus.value = false
}
//
// todolist 功能
// -------------------------------------------------------------
//
const todoList = ref([])

const getTodo = async () => {
  const response = await axios.get(`${url}/todos`, {
    headers: {
      Authorization: token.value,
    },
  })
  todoList.value = response.data.data.map((todo) => ({ ...todo, isEditing: false }))
}

const newTodo = ref('')

const addTodo = async () => {
  try {
    const todo = {
      content: newTodo.value,
    }

    await axios.post(`${url}/todos`, todo, {
      headers: {
        Authorization: token.value,
      },
    })

    alert('新增成功')
    newTodo.value = ''
    getTodo()
  } catch (error) {
    alert('新增失敗:' + error.message)
  }
}

const removeTodo = async (id) => {
  try {
    await axios.delete(`${url}/todos/${id}`, {
      headers: {
        Authorization: token.value,
      },
    })

    alert('刪除成功')
    getTodo()
  } catch (error) {
    alert('刪除失敗:' + error.message)
  }
}

const toggleEdit = (todo) => {
  if (todo.isEditing) {
    updateTodo(todo.id)
  } else {
    todo.isEditing = true
  }
}

const updateTodo = async (id) => {
  try {
    const todo = todoList.value.find((todo) => todo.id === id)

    await axios.put(`${url}/todos/${id}`, todo, {
      headers: {
        Authorization: token.value,
      },
    })

    getTodo()
    alert('編輯成功')
  } catch (error) {
    alert('編輯失敗:' + error.message)
  }
}

AOS.init()
</script>

<style>
body {
  background-color: #f8f9fa;
}

input {
  width: 70%;
}
</style>
