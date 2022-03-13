# vue+firebase 블로그 프로젝트

#app.vue
  <template>
  <!-- 상단 사이드바 -->
  <nav class="navbar navbar-expand-lg navbar-light bg-light">
    <div class="container-fluid">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0" v-if="this.userName == ''">
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
            <span class="material-icons"> account_circle </span>
          </a>
          <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
            <li><a class="dropdown-item" @click="CheckUser">로그인</a></li>
            <li><hr class="dropdown-divider" /></li>
            <li><a class="dropdown-item" id="drit1" @click="signInWithGoogle(), CheckUser()">구글로그인</a></li>
          </ul>
        </li>
      </ul>
      <ul class="navbar-nav me-auto mb-2 mb-lg-0" v-else>
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
            <span class="material-icons"> account_circle </span>
          </a>
          <ul class="dropdown-menu" aria-labelledby="navbarDropdown">
            <li>
              <a class="dropdown-item">Hello! {{ this.userName }}</a>
            </li>
            <li><hr class="dropdown-divider" /></li>
            <li><a class="dropdown-item" id="drit2" @click="signOut">로그아웃</a></li>
          </ul>
        </li>
      </ul>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0"></ul>
        <form class="d-flex">
          <input class="form-control me-2" placeholder="Search Title" v-model="searchData" />
        </form>
        <button class="btn btn-outline-success" @click="searchText">Search</button>
      </div>
    </div>
  </nav>
  <div class="jumbotron main-background" id="header">
    <!-- <img src="./assets/night.jpg" /> -->
    <h1 class="display-4">Weht's Blog</h1>
    <hr class="my-4" />
  </div>
  <SideBar />
  <router-view :fireData="fireData" :userUid="userUid"> </router-view>
  <blogFooter />
</template>

<script>
import SideBar from './components/sideBar.vue'
import blogFooter from './components/blogFooter.vue'
import { db } from './main.js'
import 'firebase/firestore'
import firebase from 'firebase/app'

export default {
  name: 'App',
  data() {
    return {
      cnt: true,
      loading: false,
      userName: '',
      fireData: [],
      searchData: '',
      userUid: '',
    }
  },
  components: {
    SideBar,
    blogFooter,
  },
  methods: {
    sideToggle() {
      var target = document.getElementById('sideBar')
      if (this.cnt == true) {
        target.style.display = 'block'
        this.cnt = false
      } else {
        target.style.display = 'none'
        this.cnt = true
      }
    },
    testFirebase() {
      db.collection('user')
        .get()
        .then((result) => {
          result.forEach((doc) => {
            console.log(doc.data())
          })
        })
    },
    signInWithGoogle() {
      var provider = new firebase.auth.GoogleAuthProvider()
      console.log(provider)
      this.loading = true
      try {
        var result = firebase.auth().signInWithPopup(provider)
        console.log(result)
      } finally {
        this.loading = false
      }
    },
    CheckUser() {
      firebase.auth().onAuthStateChanged((user) => {
        if (user) {
          localStorage.setItem('user', JSON.stringify(user))
          var saveUser = localStorage.getItem('user')
          this.userName = JSON.parse(saveUser).displayName
          this.userUid = JSON.parse(saveUser).uid
        }
      })
    },
    signOut() {
      firebase
        .auth()
        .signOut()
        .then(() => {
          alert('로그아웃되었습니다')
          this.userName = ''
        })
    },
    CheckData() {
      db.collection('post')
        .orderBy('date', 'asc')
        .get()
        .then((result) => {
          result.forEach((doc) => {
            this.fireData.push({
              title: doc.data().title,
              content: doc.data().content,
              img: doc.data().imgURL,
              date: doc.data().date,
              id: doc.id,
              storageURL: doc.data().storageURL,
              uid: this.userUid,
            })
          })
        })
    },
    searchText() {
      let data = this.fireData
      let result = data.filter((i) => new RegExp(this.searchData, 'i').test(i.title))
      console.log(result)
      this.fireData = result
    },
  },
  mounted() {
    this.CheckUser()
    this.CheckData()
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');
@import './assets/appCss.css';
</style>
