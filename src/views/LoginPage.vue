<template>
  <div class="login-container">
    <h2>{{ isLoginMode ? 'Вход' : 'Регистрация' }}</h2>
    <form @submit.prevent="handleSubmit">
      <div class="input-field">
        <label>Email:</label>
        <input type="email" v-model="email" required>
      </div>
      <div class="input-field">
        <label>Пароль:</label>
        <input type="password" v-model="password" required>
      </div>
      <div class="input-field" v-if="!isLoginMode">
        <label>Подтвердите пароль:</label>
        <input type="password" v-model="passwordConfirm" required>
      </div>
      <button type="submit">{{ isLoginMode ? 'Войти' : 'Зарегистрироваться' }}</button>
      <button type="button" @click="toggleMode">{{ isLoginMode ? 'Нужен аккаунт? Регистрация' : 'Уже есть аккаунт? Войти' }}</button>
    </form>
  </div>
</template>


<script>
import { auth, db, createUserWithEmailAndPassword, signInWithEmailAndPassword } from '../../firebase-config';
import { doc, setDoc } from 'firebase/firestore';
import router from "@/router/index.js";

export default {
  data() {
    return {
      email: '',
      password: '',
      passwordConfirm: '',
      isLoginMode: true,
      errorMessage: ''
    };
  },
  methods: {
    async handleSubmit() {
      if (this.isLoginMode) {
        try {
          await signInWithEmailAndPassword(auth, this.email, this.password);
          console.log("Вход выполнен успешно");
          await router.push('/');
        } catch (error) {
          this.errorMessage = error.message;
          console.error('Ошибка входа:', error.message);
        }
      } else {
        if (this.password !== this.passwordConfirm) {
          this.errorMessage = "Пароли не совпадают.";
          return;
        }
        try {
          const userCredential = await createUserWithEmailAndPassword(auth, this.email, this.password);
          console.log("Регистрация выполнена успешно");
          await this.createUserProfile(userCredential.user);
          await router.push('/login');
        } catch (error) {
          this.errorMessage = error.message;
          console.error('Ошибка регистрации:', error.message);
        }
      }
    },
    async createUserProfile(user) {
      try {
        await setDoc(doc(db, "users", user.uid), {
          email: user.email
        });
        console.log("Профиль пользователя успешно создан.");
      } catch (error) {
        console.error("Ошибка создания профиля пользователя:", error.message);
      }
    },
    toggleMode() {
      this.isLoginMode = !this.isLoginMode;
    }
  }
}
</script>




<style>
body, html {
  height: 100%; /* Устанавливаем высоту всей страницы */
  margin: 0; /* Убираем стандартные отступы */
  padding: 0;
  display: flex; /* Используем Flexbox */
  justify-content: center; /* Центрирование по горизонтали */
  align-items: center; /* Центрирование по вертикали */
  background-color: #f0f0f0; /* Фон всей страницы */
}

.login-container {
  color: #181818;
  width: 100%;
  max-width: 400px; /* Максимальная ширина */
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1); /* Тень вокруг формы */
  border-radius: 8px; /* Скругление углов */
  background-color: white; /* Фон формы */
}

.input-field {
  margin-bottom: 20px; /* Отступ между полями */
}

input[type="email"],
input[type="password"] {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  width: 100%;
  padding: 10px;
  border: none;
  color: white;
  background-color: #f5b000;
  cursor: pointer;
  border-radius: 4px;
  margin-top: 10px;
}

button:hover {
  background-color: #ecc849; /* Изменение цвета при наведении */
}

h2 {
  text-align: center; /* Центрирование заголовка */
}
</style>

