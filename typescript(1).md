# Vue.js Todo App 타입스크립트로 전환하기 - (1) typescript 맛보기

### typescript 설정해서 vue-cli 프로젝트 만들기 

먼저 vue-cli 세팅을 typescript로 전환해야 하는데, PWA 등 새로 추가할 feature가 있었고 기존 코드와 비교하면서 코딩하기 쉽게 하기 위해 그냥 새로 프로젝트를 생성했다. 

`vue create appname` 후 default preset 설정하지 않고 수동으로 feature 고르는 항목들 중 typescript를 선택하면 된다

프로젝트 구조를 보면 기존 javascript 프로젝트와 다른 점이 두가지 보인다:

* .js파일이 .ts로 바뀐 것

* shims-vue.d.ts, shims-tsx.d.ts 파일이 새로 생김

``` declare *module* '*.vue' {
//shims-vue.d.ts

declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

``` 
//  shims-tsx.d.ts

import Vue, { VNode } from 'vue'

declare global {
  namespace JSX {
    // tslint:disable no-empty-interface
    interface Element extends VNode {}
    // tslint:disable no-empty-interface
    interface ElementClass extends Vue {}
    interface IntrinsicElements {
      [elem: string]: any
    }
  }
}
```

shims-vue.d.ts, shims-tsx.d.ts 이 친구들은 뭐하는 파일일까? 검색해보니 전자는 IDE가 .vue로 끝나는 파일이 무슨 역할을 해야 하는지 이해하게 해주는 것 = `.vue` 싱글 파일 컴포넌트를 import하고 사용할 수 있게 해주는 것, 후자는 IDE 내의 `jsx`syntaxsupport가 JSX 스타일의 typescript 코드를 쓸수 있게 하면서, 결국은 `.tsx` 파일을 사용할 수 있게 해주는 것이라고 한다. 그냥 그 정도만 알고 있으면 될 것 같다… 

(출처: https://stackoverflow.com/questions/54622621/what-does-the-shims-tsx-d-ts-file-do-in-a-vue-typescript-project)

난 기존 자바스크립트 프로젝트의 코드를 복붙한 후 타입스크립트에 맞게 코드를 고쳐나가는 (주로 error message를 따라) 식으로 전환을 진행했다. 

먼저 타입스크립트에 대해 이론적으로 깊게 공부하고 난 후 시작한게 아니라 굉장히 중구난방식이지만 여러 코드를 다뤄보면서 점점 더 발전하지 않을까 하는 기대를…ㅎ ㅎ

## 직접 코딩하기 전에… TypeScript 기초 알아보기

typescript의 이름답게 <strong>type</strong>이 필수적으로 필요하다

javascript와의 아마 가장 중요하고 가장 큰 차이점이지 않을까

#### TypeScript는 프로그래밍 언어이자, Compiled Language <=> Interpreted Language(Js)

컴파일이 필요한 언어이며, 컴파일러가 컴파일 과정에서 type check가 이루어진다

### Static Typing 

정적 타이핑 언어는 컴파일 시간에 미리 체크 vs. 동적 타이핑 언어는 런타임에 체크 

동적 타입 언어 (Python, Javascript)는 유연하고 편리하지만 버그를 미리 잡아낼 확률이 낮다. 편하지만 나중에 고생하는 언어.

정적 타입 언어(C, Java)는 컴파일 타임에 미리 검사를 하기 때문에 실행 성능이 올라가고 미리 버그를 잡아낼 수 있다. 불편하지만 확실한 언어. 

결국 타입스크립트는 자바스크립트의 동적 타이핑에서 오는 불편함을 보완하기 위해 등장한 것! 

많은 개발자들이 타입스크립트를 좋아하는 이유도 타입스크립트가 버그지옥, 잔실수로부터 보호해주기 때문인 듯.

### Basic Types of ts 

* Boolean

* Number

  * int, double, binary 등의 개념을 모두 포함 

* String

* Array

  * `type[]`의 형태로 사용 : `number[]`
  * generic array type도 사용 가능: `Array<elemType>` (java랑 비슷하다)

* Tuple

  ``` typescript
  let x: [string, number]
  x = ["hello", 10]
  ```

* Enum (default는 0부터 시작)

  ``` typescript
  enum Color {Red, Green, Blue}
  let c: Color = Color.Green;
  
  // can manually set values 
  // enum Color {Red = 1, Green = 2, Blue = 4}
  ```

* Any : 타입이 확실하지 않거나 동적으로 결정될 때

  * Object와의 차이점? Object는 Object 클래스에 정의된 함수와 속성만 사용 가능 (좀 더 제한적)

* Void : Any 와 반대되는 개념 => 어떤 type도 가지지 않는 것 

  * void로 선언한 변수는 null만 할당할 수 있기 때문에 잘 쓰이지 않음 

* Null and Undefined

* Never : 절대 발생하지 않는 값 나타낼 때 쓰임

  * e.g. 항상 예외가 발생하거나 리턴되지 않는 함수의 리턴타입은 never 

  ```typescript
  function error(message: string): never {
      throw new Error(message);
  }
  ```

  

* Object : primitive type이 아닌 type => number, string, boolean, bigint, symbol, null, undefined가 아닌 타입 나타냄 

* Type Assertions 

  * type보다 더 구체적으로 객체나 변수의 형태를 알 때 사용한다 
  * a way to tell the compiler “trust me, I know what I’m doing."

  ```typescript
  let someValue: any = "this is a string";
  let strLength: number = (<string>someValue).length;
  // let strLength: number = (someValue as string).length; 와 같음 
  ```

  ` (<string>someValue)`, `as`를 사용해서 표현된 부분이 그것!

  "number이긴 한데, 더 구체적으론 string인 someValue의 길이야." 라고 컴파일러에게 힌트를 주는 것 

```typescript
class Animal {
  age: number,
  cry() {
    /*...*/
  }
  isDog() {
    /*...*/
  }
  isCat() {
    /*...*/
  }
}

class Dog extends Animal {
  bark(){ 
    /*...*/
  }
}

class Cat extends Animal {
  meow(){ 
    /*...*/
  }
}

function makeSound(animal: Animal){
  if(animal.isDog()) {
    animal.bark() 
    // Property 'bark' does not exist on type 'Animal'.
  }
  else if(animal.isCat()) {
    animal.meow()
    // Property 'meow' does not exist on type 'Animal'.
  }
  else {
    animal.cry()
  }
}
```

위와 같이, Animal 클래스의 선언되어있지 않은 함수를 animal 객체에서 불러내면 에러가 난다. type assertion을 사용하면 컴파일러에게 이게 Dog이야! 이게 Cat이야!라는 걸 보장해줄 수 있기 때문에 에러를 방지할 수 있다. 

```typescript
function makeSound(animal: Animal){
  if(animal.isDog()) {
    (animal as Dog).bark() 
    //(<Dog>animal)
  }
  else if(animal.isCat()) {
    (animal as Cat).meow()
    //(<Cat>animal)
  }
  else {
    animal.cry()
  }
}
```

### 





