# How_to_use_vuex

지금껏 사용했던 vuex 방식이 너무 복잡해서 간단한 방법을 찾아왔다
잡담없이 바로 들어가자

## 스토어 생성

vue-cli를 통해 vuex를 설치했다면 frontend → src 루트에 store라는 폴더가 보일것이다
여기에 modules폴더를 생성하고, 관리할 스토어를 모듈로서 하나씩 추가하는 방식으로 진행할 것이다
여기서는 userStore.js라는 이름으로 userStore를 관리해볼것이다.
modules폴더에 userStore.js 파일 생성

userStore.js

```jsx
const userStore = {
  namespaced: true,
  state: {
    userName: '이대욱'
  },
  getters: {
    GE_USER_NAME: state => state.userName
  },
  mutations: {
    MU_USER_NAME: (state, payload) => {
      state.userName = payload.userName
    }
  },
  actions: {
    AC_USER_NAME: ({ commit }, payload) => {
      commit('MU_USER_NAME', payload)
    }
  }
}

export default userStore
```

그리고 원래 store폴더에 들어있던 index.js를 수정해보자

store -> index.js

```jsx
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

import userStore from '@/store/modules/userStore.js'

const store = new Vuex.Store({
  modules: {
    userStore: userStore,
  }
})

export default store
```
modules폴더에서 만든 스토어들을 index.js로 임포트 한 뒤, 다른곳에서 사용할 때에는 
index.js에서만 꺼내간다고 생각하면 편할것이다


vue-cli로 vuex를 인스톨 했다면 뷰 인스턴스에 store가 들어있었을 것이다.
import 부분에서 경로만 맞는지 확인해보자
main.js

```jsx
import Vue from 'vue'
import App from './App.vue'
import './registerServiceWorker'
import router from './router'
import store from './store/index.js'
import vuetify from './plugins/vuetify';
import axios from 'axios'

Vue.prototype.$http = axios
Vue.config.productionTip = false

new Vue({
  router,
  store,
  vuetify,
  render: h => h(App)
}).$mount('#app')
```

# 스토어 생성은 끝났으니 본격적으로 사용법을 알아보자

App.vue에서 <script> 부분만 떼어왔다
  
App.vue

```jsx
<script>
// vuex 라이브러리에서 mapActions, mapGetters 함수를 가져온다.
import { mapActions, mapGetters } from 'vuex'

const userStore = 'userStore'

export default {
  name: 'App',
  data() {
    return {
      userName: ''
    }
  },
  computed: {
    /*
      mapGetter는 store의 getters를 가져온다.

      네임스페이스를 사용하기 때문에 키 이름을 넣는다 (userStore)

      2가지 방식으로 가져올 수 있다
      1) 이름 지정해서 가져오기
      2) getters 이름 그대로 사용해서 가져오기
    */
    // 1) 이름 지정해서 가져오기
    ...mapGetters(userStore, {
      storeUserName: 'GE_USER_NAME'
    }),

    // 2) getters 이름 그대로 사용해서 가져오기, 이걸 선호한다 
    ...mapGetters(userStore, [
      'GE_USER_NAME'
    ]),
  },
  watch: {
    // getters에 watch를 걸 수 있다
    GE_USER_NAME(val) {
      this.userName = val
    }
  },  
  created() {
    this.userName = this.GE_USER_NAME
  },
  methods: {
    /*
      mapGetter는 store의 getters를 가져온다

      네임스페이스를 사용하기 때문에 키 이름을 적는다. (userStore)

      2가지 방식으로 가져올 수 있다
      1) 이름 지정해서 가져오기
      2) getters 이름 그대로 사용해서 가져오기      

      개인의 취향이지만, getters 이름 그대로 사용하는 것을 추천.

      다른 메소드 이름으로 매핑 예를 들면, setUserName: AC_USER_NAME 하면,
      setUserName 함수가 나중에는 스토어 함수인지, 현재 파일의 함수인지 헷갈리는 경우가 있다.
    */
    ...mapActions(userStore, [
      'AC_USER_NAME'
    ]),
    // 버튼을 클릭하면 수행.
    searchName() {
      const payload = {
        userName: this.userName
      }
      // store의 userName을 변경.
      this.AC_USER_NAME(payload)
    }
  }
}
</script>
```

위의 방법으로 사용할 수 있다

---

## 새로고침 되어도 상태를 기억하기 위한 패키지

새로고침하면 스토어에 저장된 데이터가 증발하므로 새로고침해도 유지시키기 위해 아래 패키지가 필요하다

`npm i vuex-persistedstate`

frontend → src → store → index.js

```jsx
import Vue from 'vue'
import Vuex from 'vuex'
import createPersistedState from "vuex-persistedstate";

Vue.use(Vuex)

import userStore from '@/store/modules/userStore.js'
import operationStore from '@/store/modules/operationStore.js'

const store = new Vuex.Store({
  modules: {
    userStore: userStore,
    operationStore: operationStore,
  },
  plugins: [createPersistedState()],
})

export default store
```
