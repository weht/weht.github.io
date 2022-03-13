# vue+firebase 블로그 프로젝트

## app.vue
```
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
```
# Component

## wehtProfile.vue

```
  <template>
  <div class="container mt-5 mb-3">
    <div class="row">
      <div class="col-md-4"></div>
      <div class="col-md-4">
        <div class="card p-3 mb-2">
          <div class="d-flex justify-content-between">
            <div class="d-flex flex-row align-items-center">
              <div class="icon"><i class="material-icons">&#xE87C;</i></div>
              <div class="ms-2 c-details">
                <h6 class="mb-0">weht</h6>
                <span>3.12</span>
              </div>
            </div>
            <div class="badge"><span>frontend</span></div>
          </div>
          <div class="mt-5">
            <h3 class="heading">Github<br /><a href="https://github.com/weht">https://github.com/weht</a></h3>
            <div class="mt-5">
              <div class="mt-3">
                <span class="text1">vue + firebase <span class="text2">project</span></span>
              </div>
              <div class="mt-4"></div>
            </div>
            <div class="mt-4"></div>
          </div>
          <div class="mt-4"></div>
        </div>
        <div class="mt-4"></div>
      </div>
      <div class="mt-4"></div>
    </div>
    <div class="mt-5"></div>
  </div>
</template>

<script>
export default {
  name: 'wehtProfile',
}
</script>

<style></style>
```

## sidebar.vue

```
  <template>
  <div class="menu">
    <label for="expand-menu" @click="sideToggle"><div>메뉴</div></label>
    <input type="checkbox" id="expand-menu" name="expand-menu" />
    <ul id="sideBar">
      <li>
        <router-link to="/profile" class="item"><div>프로필</div></router-link>
      </li>
      <li>
        <router-link to="/write" class="item"><div>글쓰기</div></router-link>
      </li>
      <li>
        <a href="/" class="item"><div>홈 / 글목록</div></a>
      </li>
      <li>
        <a class="item" @click="goToTop"><div>TOP</div></a>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'sideBar',
  methods: {
    goToTop() {
      window.scrollTo(0, 0)
    },
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');
@import '../assets/appCss.css';
</style>
```

## postRetouch.vue

```
  <template>
  <h1 id="writeHead">블로그 글 수정</h1>
  <div class="input-group mb-3">
    <span class="input-group-text" id="inputGroup-sizing-default">제목</span>

    <input
      type="text"
      class="form-control"
      id="inputGroup-sizing-default2"
      aria-label="Sizing example input"
      aria-describedby="inputGroup-sizing-default"
      v-model="title"
    />
  </div>
  <div id="editor"></div>
  <input type="file" id="image" accept="image/*" />
  <button class="btn btn-primary" id="sub" @click="updateData">수정완료</button>
</template>

<script>
import Editor from '@toast-ui/editor'
import '@toast-ui/editor/dist/toastui-editor.css' // Editor style
import 'firebase/firestore'
import 'firebase/storage'
import 'base-64'
import firebase from 'firebase/app'
import { db } from '../main'

export default {
  name: 'editorWrite',
  data() {
    return {
      editor: null,
      localDisplayuid: '',
      title: '',
      content: '',
    }
  },
  props: {
    fireData: Array,
  },
  mounted() {
    this.checkData()
    setTimeout(() => {
      this.editor = new Editor({
        el: document.querySelector('#editor'),
        height: '500px',
        width: 'auto',
        initialEditType: 'wysiwyg',
        initialValue: this.content,
        hooks: {},
      })
    }, 900)
  },
  methods: {
    updateData() {
      const storage = firebase.storage()
      var file = document.querySelector('#image').files[0]
      var uploadImg = storage
        .ref()
        .child('image/' + file.name)
        .put(file)
      uploadImg.on(
        'state_changed',
        null,
        (error) => {
          console.error('실패사유는', error)
        },
        () => {
          uploadImg.snapshot.ref.getDownloadURL().then((url) => {
            var updateData = {
              content: this.editor.getHTML(),
              date:
                new Date().getFullYear() +
                '-' +
                (new Date().getMonth() + 1) +
                '-' +
                new Date().getDate() +
                ' ' +
                new Date().getHours() +
                ':' +
                new Date().getMinutes(),
              title: this.title,
              imgURL: url,
            }
            console.log('업로드된 경로는', url)
            db.collection('post')
              .doc(this.fireData[this.$route.params.id].id)
              .update(updateData)
              .then(() => {
                alert('업데이트에 성공했습니다')
                window.location.href = '/'
              })
              .catch((err) => console.log(err))
          })
        }
      )
    },
    checkData() {
      setTimeout(() => {
        this.content = this.fireData[this.$route.params.id].content
        this.title = this.fireData[this.$route.params.id].title
      }, 900)
    },
    onEditorChange() {
      this.editor.setHTML(this.content)
    },
  },
}
</script>

<style></style>
```

## mainPage.vue
```
  <template>
  <div class="mainPosition">
    <div class="mainTitle">Home / List</div>
    <div class="container">
      <div class="row">
        <div class="col-6" v-for="(a, i) in fireData" :key="i">
          <div class="card">
            <router-link :to="{ path: '/detail/' + i }">
              <div class="image-box">
                <img :src="fireData[i].img" alt="https://via.placeholder.com/600" />
              </div>
              <div class="card-body">
                <h5 class="card-title">{{ fireData[i].title }}</h5>
                <p class="card-text">{{ fireData[i].date }}</p>
              </div>
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { db } from '../main.js'
import 'firebase/firestore'

export default {
  name: 'mainPage',
  data() {
    return {}
  },
  props: {
    fireData: Array,
  },
  methods: {
    mainData() {
      db.collection('post')
        .get()
        .then((result) => {
          result.forEach((doc) => {
            console.log(doc.data())
          })
        })
    },
  },
}
</script>

<style>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');
@import '../assets/appCss.css';
</style>
```

## blogWrite.vue

```
  <template>
  <h1 id="writeHead">블로그 글 작성</h1>
  <div class="input-group mb-3">
    <span class="input-group-text" id="inputGroup-sizing-default">제목</span>

    <input
      type="text"
      class="form-control"
      id="inputGroup-sizing-default2"
      aria-label="Sizing example input"
      aria-describedby="inputGroup-sizing-default"
      v-model="title"
    />
  </div>
  <div id="editor"></div>
  <input type="file" id="image" accept="image/*" />
  <button class="btn btn-primary" id="sub" @click="subData">작성완료</button>
</template>

<script>
import Editor from '@toast-ui/editor'
import '@toast-ui/editor/dist/toastui-editor.css' // Editor style
import 'firebase/firestore'
import 'firebase/storage'
import 'base-64'
import firebase from 'firebase/app'
import { db } from '../main'

export default {
  name: 'editorWrite',
  data() {
    return {
      editor: null,
      localDisplayuid: '',
      title: '',
    }
  },
  mounted() {
    this.editor = new Editor({
      el: document.querySelector('#editor'),
      height: '500px',
      width: 'auto',
      initialEditType: 'wysiwyg',
      hooks: {},
    })
  },
  methods: {
    subData() {
      console.log(this.editor.getHTML())
      const storage = firebase.storage()
      var file = document.querySelector('#image').files[0]
      var uploadImg = storage
        .ref()
        .child('image/' + file.name)
        .put(file)
      uploadImg.on(
        'state_changed',
        null,
        (error) => {
          console.error('실패사유는', error)
        },
        () => {
          uploadImg.snapshot.ref.getDownloadURL().then((url) => {
            console.log('업로드된 경로는', url)
            var saveUser = localStorage.getItem('user')
            this.localDisplayuid = JSON.parse(saveUser).uid
            var postData = {
              content: this.editor.getHTML(),
              date:
                new Date().getFullYear() +
                '-' +
                (new Date().getMonth() + 1) +
                '-' +
                new Date().getDate() +
                ' ' +
                new Date().getHours() +
                ':' +
                new Date().getMinutes(),
              title: this.title,
              imgURL: url,
              storageURL: 'image/' + file.name,
              uid: this.localDisplayuid,
            }
            db.collection('post')
              .add(postData)
              .then(() => {
                alert('등록에 성공했습니다')
                window.location.href = '/'
              })
              .catch((err) => {
                console.log(err)
              })
          })
        }
      )
    },
  },
}
</script>

<style></style>
```

## blogFooter.vue

```
  <template>
  <footer class="site-footer">
    <div class="container">
      <div class="row">
        <div class="footer-text">
          <h6>About</h6>
          <p class="text-justify">이 사이트는 vue와 firebase로 만들어져 있습니다</p>
        </div>
      </div>
      <hr />
    </div>
    <div class="container">
      <div class="row">
        <div class="col-md-8 col-sm-6 col-xs-12">
          <p class="copyright-text">This site was created in 2022 Project start date 2.28 ~ 3.12</p>
        </div>
      </div>
    </div>
  </footer>
</template>

<script>
export default {
  name: 'blogFooter',
}
</script>

<style>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');
@import '../assets/appCss.css';
</style>
```

## blogDetail.vue
```
  <template>
  <div class="input-group mb-3">
    <span class="input-group-text" id="inputGroup-sizing-default">제목</span>
    <input
      type="text"
      class="form-control"
      id="inputGroup-sizing-default2"
      aria-label="Sizing example input"
      aria-describedby="inputGroup-sizing-default"
      v-model="title"
      readonly
    />
  </div>
  <label for="exampleFormControlTextarea1" class="form-label">썸네일용 사진</label>
  <div class="col-6">
    <div class="thumbnail-box">
      <div class="image-box">
        <img :src="fireData[this.$route.params.id].img" alt="" />
      </div>
    </div>
  </div>
  <div class="mb-3">
    <label for="exampleFormControlTextarea1" class="form-label">내용</label>
    <p v-html="content"></p>
  </div>
  <router-link :to="{ path: '/retouch/' + this.$route.params.id }"
    ><button class="btn btn-secondary" id="retouch">수정</button></router-link
  >
  <button class="btn btn-danger" id="delete" @click="deleteData">삭제</button>
  <input type="hidden" id="hiddenNumber" v-model="this.$route.params.id" />
</template>

<script>
import 'firebase/firestore'
import 'firebase/storage'
import 'base-64'
import firebase from 'firebase/app'
import { db } from '../main'

export default {
  name: 'blogDetail',
  props: {
    fireData: Array,
    userUid: String,
  },
  data() {
    return {
      viewer: null,
      title: '',
      content: '',
    }
  },
  mounted() {
    this.checkData()
  },
  methods: {
    checkData() {
      setTimeout(() => {
        this.content = this.fireData[this.$route.params.id].content
        this.title = this.fireData[this.$route.params.id].title
      }, 900)
    },
    deleteData() {
      db.collection('post')
        .doc(this.fireData[this.$route.params.id].id)
        .delete()
        .then(() => {
          const storage = firebase.storage()
          storage.ref().child(this.fireData[this.$route.params.id].storageURL).delete()
          alert('현재 포스트 삭제를 완료했습니다')
          window.location.href = '/'
        })
    },
    hideButton() {
      var saveUser = localStorage.getItem('user')
      if (this.userUid != JSON.parse(saveUser).uid) {
        document.querySelector('#retouch').style.display = 'none'
        document.querySelector('#delete').style.display = 'none'
      }
    },
  },
}
</script>

<style></style>
```

# 끝
