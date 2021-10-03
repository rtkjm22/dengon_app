<template>
  <section>
    <div class="alert h6 text-right" @click="doLogin">
      [login: {{ data.user != null ? data.user.displayName : "---" }} ]
    </div>
    <h2>{{ data.title }}</h2>
    <p class="h5">{{ data.message }}</p>

    <div class="alert alert-primary" v-if="data.user">
      <div class="form-group text-left">
        <label class="h5">Message</label>
        <div class="form-row">
          <div class="col-10">
            <input v-model="data.msg" class="form-control" />
          </div>
          <button @click="add" class="btn btn-primary col-2">投稿</button>
        </div>
      </div>
      <h3 class="my-3">Messages</h3>
      <ul
        v-for="(item, key) in data.fire_data"
        :key="key"
        class="list-group text-left"
      >
        <li class="list-group-item">
          <div class="h5">{{ item.msg }}</div>
          <div class="small text-light">
            {{ item.user }} ( {{ item.posted }} )
          </div>
        </li>
      </ul>
    </div>

    <div v-else>
      <div class="alert alert-warning">※現在、ログインされていません。</div>
    </div>
  </section>
</template>

<script>
import { onMounted, reactive } from "vue";
import { initializeApp } from "firebase/app";
import {
  getDatabase,
  ref,
  onValue,
  query,
  orderByKey,
  limitToLast,
  set,
} from "firebase/database";

import {
  getAuth,
  signInWithPopup,
  GoogleAuthProvider,
  signOut,
} from "firebase/auth";

const firebaseConfig = {
  apiKey: import.meta.env.VITE_API_KEY,
  authDomain: import.meta.env.VITE_AUTH_DOMAIN,
  databaseURL: import.meta.env.VITE_DATABASE_URL,
  projectId: import.meta.env.VITE_PROJECT_ID,
  storageBucket: import.meta.env.VITE_STORAGE_BUCKET,
  messagingSenderId: import.meta.env.VITE_MESSAGING_SENDER_ID,
  appId: import.meta.env.VITE_APP_ID,
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);
const provider = new GoogleAuthProvider();
const auth = getAuth();

export default {
  setup(props) {
    const data = reactive({
      title: "Board",
      message: "ミニ伝言板。最新の投稿を表示します。",
      user: null,
      msg: "",
      num_per_page: 10,
      fire_data: {},
    });

    // Login process
    const login = () => {
      signInWithPopup(auth, provider).then((result) => {
        const credential = GoogleAuthProvider.credentialFromResult(result);
        const token = credential.accessToken;
        const user = result.user;
        data.user = user;
        data.message = "ログインしました。";
        const getdb = query(
          query(ref(db, "/board"), orderByKey()),
          limitToLast(data.num_per_page)
        );
        onValue(getdb, (snapshot) => {
          if (auth.currentUser != null) {
            const persons = snapshot.val();
            let arr = [];
            for (let item in persons) {
              arr.unshift(persons[item]);
            }
            data.fire_data = arr;
          } else {
            console.log("エラーが発生しました。");
            data.fire_data = {};
          }
        });
      });
    };

    //Logout process
    const logout = () => {
      signOut(auth).then(() => {
        auth.currentUser == null;
        data.user = null;
        data.message = 'ログアウトしています。'
        console.log('ログアウトしました。');
      });
    };

    //Toggle function between login/logout
    const doLogin = () => {
      if (auth.currentUser === null) {
        login();
      } else {
        logout();
      }
    };

    const add = () => {
      if (auth.currentUser === null) {
        alert("ログインしないと投稿できません。");
        return;
      }

      let d = new Date();
      let dstr = `${d.getFullYear()}-${d.getMonth()}-${d.getDate()} ${d.getHours()}:${d.getMinutes()}:${d.getSeconds()}`;
      let userId = d.getTime();
      let obj = {
        msg: data.msg,
        user: data.user.displayName,
        posted: dstr,
      };
      if (obj.msg !== '') {
        set(ref(db, 'board/' + userId), obj)
        data.msg = ''
        data.message = '投稿しました。'
      } else {
        data.message - 'メッセージを入力してください。'
      }
    };

    onMounted(() => {
    if (auth.currentUser === null) {
      login()
    }
    });
    return {
      data,
      login,
      logout,
      doLogin,
      add,
    };
  },
};
</script>
