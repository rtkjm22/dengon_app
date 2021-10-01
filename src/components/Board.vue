<template>
  <section>
    <div class="alert h6 text-right"
      @click="doLogin">
      [login: {{ data.user !=null ? data.user.displayName : '---' }} ]
      </div>
      <h2>{{ title }}</h2>
      <p class="h5">{{ data.message }}</p>

      <div class="alert alert-primary" v-if="data.user">
        <div class="form-group text-left">
          <label class="h5">Message</label>
          <div class="form-row">
            <div class="col-10">
              <input v-model="data.msg" class="form-control">
            </div>
            <button @click="add" class="btn btn-primary col-2">投稿</button>
          </div>
        </div>
        <h3 class="my-3">Messages</h3>
        <ul v-for="(item, key) in data.fire_data" :key="key" class="list-group text-left">
          <li class="list-group-item">
            <div class="h5">{{ item.msg }}</div>
            <div class="small text-light">{{ item.user }} ( {{ item.posted }} )</div>
          </li>
        </ul>
      </div>

      <div v-else>
        <div class="alert alert-warning">※現在、ログインされていません。</div>
      </div>

  </section>
</template>

<script lang="ts">
import { onMounted, reactive } from 'vue'
import { initializeApp } from "firebase/app";
import { getDatabase, ref, onValue } from "firebase/database";
import { getAuth, signInWithPopup ,GoogleAuthProvider } from "firebase/auth";

const firebaseConfig = {
  
}

const app = initializeApp(firebaseConfig)
const db = getDatabase(app)
const provider = new GoogleAuthProvider()
const auth = getAuth()

export default {
  setup(props) {
    const data = reactive({
      title: 'Board',
      message: 'ミニ伝言板。最新の投稿を表示します。',
      user: null,
      msg: '',
      num_per_page: 10,
      fire_data: {},
    })

    // Login process
    const login = () => {
      signInWithPopup(auth, provider)
        .then((result) => {
          const credential = GoogleAuthProvider.credentialFromResult(result)
          console.log(credential)
          const token = credential.accessToken
          const user = result.user
          data.user = user
          data.message = 'ログインしました。'
          const getdb = ref(db, '/board')
          onValue(getdb, (snapshot) => {
            const persons = snapshot.val()
            console.log(persons)
          })
        })
    }
    return {
      data, login,
    }

  }
}
</script>
