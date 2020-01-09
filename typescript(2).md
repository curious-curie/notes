# Vue.js Todo App 타입스크립트로 전환하기 - (2) Developing Components w/ Typescript 

자바스크립트로 쓴 뷰 컴포넌트를 타입스크립트로 바꾸는 방법엔 크게 Vue.extend를 이용하는 최신의 방법과 vue-class-component를 이용해서 클래스 스타일의 컴포넌트를 쓰는 두 가지 방법이 있다. 

### Vue.extend 

기존에 자바스크립트로 정의한 컴포넌트를 Vue.extend()로 감싸는 방식. 최신의 방식이고 간단하다. 자바스크립트에서 하던 방식 그대로 컴포넌트를 정의하면 되고, type을 사용할 수 있다. 

그러나 Vuex와 함께 사용할 때 치명적이라고 느껴진 단점은 컴포넌트 내에서 맵핑헬퍼를 사용할 수 없다는 것… 

```typescript
methods: {
  ...mapActions(['addTodo']),
  addTodo () {
    this.addTodo()
    this.newTodo = ''
  }
}
```

![image-20200105223336304](/Users/curieyoo/Library/Application Support/typora-user-images/image-20200105223336304.png)

이런식으로 methods에 mapActions를 정의한 후  'addTodo'를 사용하면 함수가 존재하지 않는다는 식의 에러가 뜬다.

그래서 action을 dispatch하기 위해서 mapActions를 사용하지 못하고 this.$store.dispatch를 사용해야한다. mapState나 mapGetters도 마찬가지... 이런 치명적인 단점 때문에 나는 클래스 스타일 컴포넌트를 사용했다. 

+) Vue.extend의 또 다른 문제점

Todo type으로 된 배열을 부모 컴포넌트로부터 props로 받는 경우를 생각해보자.

그 경우 컴포넌트에 이런 식으로 type을 선언하면, 'Todo'라는 타입을 선언해야하는데 넌 value를 쓰고있어!하는 에러가 난다. 내가 type을 선언한게 아닌, type이라는 자리에 Todo[]를 값으로 넣어줬다고 인식하는 것이다. primitive types로 선언하면 이런 오류가 안 난다. 

```typescript
props: {
  todos: {
    type: Todo[],
    required: true,
    default: []
  }
}
```

```typescript
interface Todo {
	title: string;
	completed: true;
}
```

이것을 해결하기 위해선 Object를 내가 사용하고자 하는 Interface를 리턴하는 함수로 캐스팅하면 된다. 

```typescript
type ComplexObjectInterface = {
  testProp: string
  modelName: number
}
export default Vue.extend({
  props: {
    propExample: {
      type: Object as () => ComplexObjectInterface
    }
  } 
```

왜 이렇게하면 오류가 해결되는 걸까?

`type: someObject`처럼 props의 타입을 선언하면, someObject의 constructor을 넘겨주는 꼴이 된다. 그러나 TypeScript의 인터페이스는 런타임에만 사용 가능해지기 때문에 compile시에는 컴파일러가 인식하지 못한다. 그래서 결국 컴파일러가 제대로 인식할 타입을 넘겨주지 못하기 때문에 컴파일 시에 에러가 나는 것이다. 

그래서 이런 식으로 `Object as () => ComplexObjectInterface`함수를 넘겨주면 typescript 타이핑이 자체적으로 타입에 넘겨진 함수가 인터페이스 객체를 리턴할 것이라고 여기기 때문에 에러 없이 실행이 가능한 것이다. 

어렵지만 해결 방법은 있었다. 클래스 스타일 컴포넌트를 택한 후에야 알아버렸다. 

(출처: https://frontendsociety.com/using-a-typescript-interfaces-and-types-as-a-prop-type-in-vuejs-508ab3f83480)



### 클래스 스타일 컴포넌트

TypeScript decorator인 vue-class-component(https://github.com/vuejs/vue-class-component)를 사용한다. 클래스 형식의 컴포넌트를 쓸 수 있게 해준다. 보통 vue-property-decorator(https://github.com/kaorun343/vue-property-decorator)와 함께 쓰인다. 

```typescript
import Vue from 'vue'
import Component from 'vue-class-component'

@Component({
  props: {
    propMessage: String
  }
})
export default class App extends Vue {
  // initial data
  msg = 123

  // use prop values for initial data
  helloMsg = 'Hello, ' + this.propMessage

  // lifecycle hook
  mounted () {
    this.greet()
  }

  // computed
  get computedMsg () {
    return 'computed ' + this.msg
  }

  // method
  greet () {
    alert('greeting: ' + this.msg)
  }
}
```

`@Component` 데코레이터는 클래스가 Vue 컴포넌트임을 나타내고, props, methods 등 모든 컴포넌트 옵션을 사용할 수 있다.

`…mapState`, `…mapActions`를 썼다. 그런데 또 하나의 문제점이랄까 불편한 점은 @Component 내에서 `mapActions`를 사용해서 받아온 메서드를 class 내에서 함수로 사용할 수 없었다는 것

```typescript
@Component({
  components: {
    Todo
  },
  computed: {
    ...mapState('todos', [
      'todos',
      'newTodo'
    ])
  },
  methods: {
    ...mapActions('todos', [
      'clearNewTodo',
      'addTodo',
      'loadTodos',
      'toggleTodo',
      'deleteTodo'
    ])
  }
})

```

```typescript
export default class TodoList extends Vue {
  public todos!: TodoConfig[]
  public filters!: object

  selectedFilter = 'all';
  created () {
    this.$store.dispatch('todos/loadTodos', db.collection('todos'))
  }
  addTodoItem () {
    this.$store.dispatch('todos/addTodo')
    this.$store.dispatch('todos/clearNewTodo')
  }

	@Watch('$route', { immediate: true, deep: true })
  onUrlChange (to: any) {
    this.selectedFilter = to.params.filter
  }
}
```

이런식으로 `@Component` 내에서 mapActions를 통해 함수를 받아왔어도 그걸 class 내의 인스턴스 메소드 내에서 사용할 수는 없고, `this.$store.dispatch`을 통해 접근해야한다. 

Watch는 데코레이터 `@Watch`를 사용해서 클래스 내부에서 구현할 수 있었다

