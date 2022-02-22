# ReactJS 이해하기 

### reactjs app 생성하기 

```shell
npx create-react-app my-app
cd my-app
npm start
````

### 기본 게임 App 만들어보기 

- [게임 App 만들기](https://ko.reactjs.org/tutorial/tutorial.html#completing-the-game)


### Vitual Dom 

![JPA 기본 계층도](https://github.com/keepinmindsh/tech-course/blob/9816b6eec68eeba9a1a3d1f0a6367bcd3c085400/assets/reactRendering.png)

React.js는 자바스크립트 내에 DOM Tree와 같은 구조체를 VIRTUAL DOM으로 갖고 있습니다. 다시 그릴 때는 그 구조체의 전후 상태를 비교하여 변경이 필요한 최소한의 요소만 실제 DOM에 반영합니다. 따라서 무작위로 다시 그려도 변경에 필요한 최소한의 DOM만 갱신되기 때문에 빠르게 처리할 수 있습니다.  


Virtual DOM 은 DOM 차원에서의 더블 버퍼링이랑 다름이 없습니다. 변화가 일어나면 그것을 오프라인 DOM 트리에 적용시키는데, 이 DOM 트리는 렌더링도 되지 않기때문에 연산 비용이 적다. 연산이 끝나고나면 그 최종적인 변화를 실제 DOM 에 던져주는 것입니다.  그러면, 레이아웃 계산과 리렌더링의 규모는 커지겠지만, 딱 한번만 연산이 일어나게 됩니다.
바로 이렇게, 하나로 묶어서 적용시키는것이, 연산의 횟수를 줄일 수 있는 방법입니다. 

### Props 

Props는 프로퍼티 용어의 약어 입니다. props는 컴포넌트에서 전달해서 보관하길 원하는 데이터 입니다. 즉, 컴포넌트 내에서 보관되면, 이 데이터는 수정되지 않고 보존되어야 하는 법칙이 성립된다. 만약 props의 값을 변경하고자 할 때는 컴포넌트 내부가 아닌, 부모 컴포넌트에서 이에 대한 부분이 변경되어야 합니다.

- props는 읽기 전용입니다.
- 모든 React 컴포넌트는 자신의 props를 다룰 때 반드시 순수 함수 처럼 동작해야 합니다.
- props의 이름은 사용될 context가 아닌 컴포넌트 자체의 관점에서 짓는 것을 권장합니다.

props는 React에서는 사용자가 컴포넌트에서 전달해서 보관하길 원하는 데이터입니다. 즉, 컴포넌트 내에서 데이터가 보관되면. 이 데이터는 수정되지 않고 보존되어야 하는 법칙이 성립됩니다.

```javascript

// App.js
import React from 'react';
import Hello from './Hello';

const App = () => {
  return (
    <Hello name="react" color="red"/>
  );
}

export default App;

// Hello.js
import React from 'react';

const Hello = ({ color, name }) => {
  return <div style=>안녕하세요 {name}</div>
}

export default Hello;

```