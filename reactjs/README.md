# ReactJS 이해하기 

### reactjs app 생성하기 

```shell
npx create-react-app my-app
cd my-app
npm start
````

### 기본 게임 App 만들어보기 

- [게임 App 만들기](https://ko.reactjs.org/tutorial/tutorial.html#completing-the-game)


### Props 와 State 

Props는 프로퍼티 용어의 약어 입니다. props는 컴포넌트에서 전달해서 보관하길 원하는 데이터 입니다. 즉, 컴포넌트 내에서 보관되면, 이 데이터는 수정되지 않고 보존되어야 하는 법칙이 성립된다. 만약 props의 값을 변경하고자 할 때는 컴포넌트 내부가 아닌, 부모 컴포넌트에서 이에 대한 부분이 변경되어야 합니다.

- props는 읽기 전용입니다.
- 모든 React 컴포넌트는 자신의 props를 다룰 때 반드시 순수 함수 처럼 동작해야 합니다.
- props의 이름은 사용될 context가 아닌 컴포넌트 자체의 관점에서 짓는 것을 권장합니다.


