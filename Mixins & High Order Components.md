# Mixins & High Order Components

## Mixin

여러 컴포넌트 간 공통으로 사용하고 있는 로직, 기능들을 재사용하는 방법

```js
var HelloMixins = {
  // 컴포넌트 옵션 (data, methods, created 등)
};

new Vue({
  mixins: [HelloMixins]
})
```



### 예시

DialogMixin 정의 

```js
var DialogMixin = {
  data() {
    return {
      dialog: false
    }
  },
  methods: {
    showDialog() {
      this.dialog = true;
    },
    closeDialog() {
      this.dialog = false;
    }
  }
};
```

LoginForm 컴포넌트에 Mixin 주입

```html
<!-- LoginForm.vue -->
<script>
import { DialogMixin } from './mixins.js';

export default {
  // ..
  mixins: [ DialogMixin ]
  methods: {
    submitForm() {
      axios.post('login', {
        id: this.id,
        pw: this.pw
      })
      .then(() => this.closeDialog())
      .catch((error) => new Error(error));
    }
  }
}
</script>
```



## High Order Component

HOC: 컴포넌트의 로직(코드)를 재사용하기 위한 기술 

```html
<!-- ProductList.vue -->
<template>
  <section>
    <ul>
      <li v-for="product in products">
        ...
      </li>
    </ul>
  </section>
</template>

<script>
import bus from './bus.js';

export default {
  name: 'ProductList',
  mounted() {
    bus.$emit('off:loading');
  },
  // ...
}
</script>
```

```html
<!-- UserList.vue -->
<template>
  <div>
    <div v-for="product in products">
      ...
    </div>
  </div>
</template>

<script>
import bus from './bus.js';

export default {
  name: 'UserList',
  mounted() {
    bus.$emit('off:loading');
  },
  // ...
}
</script>
```

ProductList 컴포넌트와 UserList 컴포넌트에서 공통으로 사용되는 코드를 볼 수 있다. name`은 컴포넌트의 이름을 정의하고, `mounted()`에서 이벤트 버스를 사용해서 로딩 상태를 완료시킨다.

```js
name: '컴포넌트 이름',
mounted() {
  bus.$emit('off:loading');
},
```

이렇게 컴포넌트끼리 반복되는 패턴이 있을 때 반복되는 코드를 줄이기 위해 HOC를 활용할 수 있다.

### 하이 오더 컴포넌트 사용하기 

```js
// CreateListComponent.js
import bus from './bus.js'
import ListComponent from './ListComponent.vue';

export default function createListComponent(componentName) {
  return {
    name: componentName,
    mounted() {
      bus.$emit('off:loading');
    },
    render(h) {
      return h(ListComponent);
    }
  }
}
```

```js
render(h) {
  return h(ListComponent);
}
```
(이 부분에서 h는 createElement의 alias로 쓰였다.`render: h => h(App)` 이 때 처럼…  )

CreateListComponent라는 하이 오더 컴포넌트에 반복적인 코드들을 미리 정의했다. 얘네를 기존 컴포넌트들에 어떻게 사용하면 될까? 

```js
// router.js
import createListComponent from './createListComponent.js';

new VueRouter({
  routes: [
    {
      path: 'products',
      component: createListComponent('ProductList')
    },
    {
      path: 'users',
      component: createListComponent('UserList')
    },
    ...
  ]
})
```

위 예시에서는 `router.js` 파일에 하이 오더 컴포넌트를 임포트 하여 컴포넌트를 정의할 때 공통 로직이 포함되도록 했다. 이렇게 각 컴포넌트의 이름만 정의를 해주면 컴포넌트의 공용 로직인 `mounted()`와 `name`을 가지고 컴포넌트가 생성된다. 

```js
// App.vue 
import createListComponent from './createListComponent.js';
import ProductList from './ProductList'

const ProductListComponent = createListComponent('UserList')
```

이런식으로 `App.vue`나 컴포넌트의 상위 파일에서 사용할 수도 있을 것 같다.



https://medium.com/bethink-pl/higher-order-components-in-vue-js-a79951ac9176

https://medium.com/bethink-pl/do-we-need-higher-order-components-in-vue-js-87c0aa608f48