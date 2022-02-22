# ReactJS 이해하기 

### reactjs app 생성하기 

```shell
npx create-react-app my-app
cd my-app
npm start
````

## 기본 게임 App 만들어보기 

- [게임 App 만들기](https://ko.reactjs.org/tutorial/tutorial.html#completing-the-game)


## Vitual Dom 

![JPA 기본 계층도](https://github.com/keepinmindsh/tech-course/blob/9816b6eec68eeba9a1a3d1f0a6367bcd3c085400/assets/reactRendering.png)

React.js는 자바스크립트 내에 DOM Tree와 같은 구조체를 VIRTUAL DOM으로 갖고 있습니다. 다시 그릴 때는 그 구조체의 전후 상태를 비교하여 변경이 필요한 최소한의 요소만 실제 DOM에 반영합니다. 따라서 무작위로 다시 그려도 변경에 필요한 최소한의 DOM만 갱신되기 때문에 빠르게 처리할 수 있습니다.  


Virtual DOM 은 DOM 차원에서의 더블 버퍼링이랑 다름이 없습니다. 변화가 일어나면 그것을 오프라인 DOM 트리에 적용시키는데, 이 DOM 트리는 렌더링도 되지 않기때문에 연산 비용이 적다. 연산이 끝나고나면 그 최종적인 변화를 실제 DOM 에 던져주는 것입니다.  그러면, 레이아웃 계산과 리렌더링의 규모는 커지겠지만, 딱 한번만 연산이 일어나게 됩니다.
바로 이렇게, 하나로 묶어서 적용시키는것이, 연산의 횟수를 줄일 수 있는 방법입니다.

## JSX 

- 표현식
- JSX는 주입 공격을 방지합니다.
- 기본적으로 React DOM은 JSX에 삽입된 모든 값을 렌더링 하기 전에 이스케이프하므로, 애플리케이션에서 명시적으로 작성되지 않은 내용은 주입되지 않습니다. 모든 항목은 렌더링 되기 전에 문자열로 변환됩니다.
- JSX는 객체를 표현합니다.

- JSX 속성 요소 

```javascript 

accept acceptCharset accessKey action allowFullScreen allowTransparency alt
async autoComplete autoFocus autoPlay capture cellPadding cellSpacing challenge
charSet checked classID className colSpan cols content contentEditable
contextMenu controls coords crossOrigin data dateTime default defer dir
disabled download draggable encType form formAction formEncType formMethod
formNoValidate formTarget frameBorder headers height hidden high href hrefLang
htmlFor httpEquiv icon id inputMode integrity is keyParams keyType kind label
lang list loop low manifest marginHeight marginWidth max maxLength media
mediaGroup method min minLength multiple muted name noValidate nonce open
optimum pattern placeholder poster preload radioGroup readOnly rel required
reversed role rowSpan rows sandbox scope scoped scrolling seamless selected
shape size sizes span spellCheck src srcDoc srcLang srcSet start step style
summary tabIndex target title type useMap value width wmode wrap

```


## Props 

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

## State

React 컴포넌트는 컴포넌트의 상태를 저장할 수 있습니다. 
props와의 차이점이라고 하면, state는 컴포넌트의 내부에 존재하고 있기 때문에, 상태 값 변경이 가능하다는 것입니다.  
상태 관리 메소드를 통해서 state 값을 변경해줄 수 있습니다. 

- React Hook 활용 방식 

```javascript

import {useState, useEffect} from 'react'

const Register = ({router: {query}}) => {
    const [businessType, setBusinessType] = useState<any>([]);

    useEffect(() => {
        axios.get(hostUrl + '/codes?codeType=BUSINESS_TYPE')
            .then(res => {
                setBusinessType(res.data);
            });
    }, [])

    const onRegisterFormHandler = (name: string, event: ChangeEvent) => {
        registerForm.task[name] = event.target.value;

        setRegisterForm(registerForm);
    }

    return (
         <div className="col" >
            <div className="form-group row">
                <label htmlFor="colFormLabelSm"
                        className="col-sm-4 col-form-label col-form-label-sm text-end">* 업무</label>
                <div className="col-sm-8">
                    <select className="form-control" defaultValue={registerForm.task.businessType} onChange={(event) => { onRegisterFormHandler("businessType", event)}} >
                        {
                            businessType.map((item: { code: string ; value: string; }, index: Key | null | undefined) => <option key={index} value={item.code} >{item.value}</option>)
                        }
                    </select>
                </div>
            </div>
        </div>
    )

})

```

- React Component - State


```javascript

import React, { Component } from 'react';

class CounterHandler extends Component {
  state = {
    count: 0
  }

  addCount = () => {
    this.setState({
      count: this.state.count + 1
    });
  }

  reduceCount = () => {
    this.setState({
      count: this.state.count - 1
    });
  }

  render() {
    return (
      <div>
        <h1>카운터</h1>
        <div>값: {this.state.count}</div>
        <button onClick={this.addCount}>+</button>
        <button onClick={this.reduceCount}>-</button>
      </div>
    );
  }
}

export default CounterHandler;

```