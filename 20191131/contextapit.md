# ContextApi with React hook

## What Context

context를 이용하면 단계마다 일일이 props를 넘겨주지 않고도 컴포넌트 트리 전체에 데이터를 제공할 수 있음.

일반적인 React 애플리케이션에서 데이터는 위에서 아래로 (즉, 부모로부터 자식에게) props를 통해 전달되지만, 애플리케이션 안의 여러 컴포넌트들에 전해줘야 하는 props의 경우 이 과정이 번거로울 수 있음.

context를 이용하면, 트리 단계마다 명시적으로 props를 넘겨주지 않아도 많은 컴포넌트가 이러한 값을 공유할 수 있음 

context를 사용하면 redux,mobx 에서의 처럼 상태관리를 한 곳에서 할 수 있음.

context의 주된 용도는 다양한 레벨에 네스팅된 많은 컴포넌트에게 데이터를 전달하는 것. 

context를 사용하면 컴포넌트를 재사용하기가 어려워지므로 꼭 필요할 때만 써야함.

[이유는 여기를 참조](https://ko.reactjs.org/docs/context.html)

* 우리 프로젝트를 예로 들자면 host-page에서 host의 정보 즉 hostID, profile image, email, name 등등의 정보는 host page 내의 전체적인 컴포넌트 안에서 사용될 여지가 있기 때문에 전역으로 context 설정해서 사용하는게 편해서 사용.

## ContextApi 사용

### 1. create Context

``` javascript
//src/libs/hostContext.js
import React from "react";

const HostContext = React.createContext({});
const HostProvider = HostContext.Provider;

export { HostContext, HostProvider };
```

* 리엑트 hook에서는 Consumer를 따로 사용하지 않기 때문에 정의하지 않음.

* Context 오브젝트에 포함된 React 컴포넌트인 Provider는 context를 구독하는 컴포넌트들에게 context의 변화를 알리는 역할


### 2. Providing Context

Provider를 Context를 사용할 컴포넌트로 감싼다. host app 전체에서 사용하기 때문에 우리프로젝트에선 app.js를 감쌌음

``` javascript
//src/App/App.js
return (
		<HostProvider value={hostInfo}>
			<div className="App">
				<Header />
				<Nav />
				{modal && <NewPollModal />}
				<Content event={event} />
			</div>
		</HostProvider>
	);
```
* Provider의 value를 통해 하위 컴포넌트들에게 context를 제공 할 수있음

### 3. Consuming Context

Provider로 감싼 컴포넌트의 하위 컴포넌트에서 context를 참조하여 사용할 수 있음
react hook에서는 간단하게 consumer로 감싸지 않고 useContext를 사용하여 이를 참조 가능
useContext를 사용하면 상위에서 가장 가까운 짝을 이루는 provider를 찾아 그에 해당하는 context를 참조.

``` javascript
//src/component/HeaderAccountAvata.j
import { makeStyles } from "@material-ui/core";
import Avatar from "@material-ui/core/Avatar";
import React, { useContext } from "react";
import { HostContext } from "../libs/hostContext";

function HeaderAccountAvata({ userName }) {
	const host = useContext(HostContext);
	console.log(host);
	const useStyles = makeStyles({
		headerAvatar: {
			backgroundColor: "#FFF",
			color: "#212529",
			margin: "0.5rem",
			"&:hover": {
				color: "#FFF",
				backgroundColor: "#69747f",
			},
		},
		avatarImage: {
			width: "2.5rem",
			height: "2.5rem",
			borderRadius: "15px",
		},
	});
	const classes = useStyles();
	const inner = userName.slice(0, 1);

	return (
		<Avatar className={classes.headerAvatar}>
			<img className={classes.avatarImage} src={host.image}></img>
		</Avatar>
	);
}

export default HeaderAccountAvata;

```
* 우리 프로젝트에서 헤더부분에 host profile image를 뜨위기 위해 사용 host에 대한 정보는 app이 시작될 때 서버로 부터 받고 이는 createModal,hostModal 등 다양한 곳에서 사용하기 때문에 context로 사용.

[react hook with context api](https://www.taniarascia.com/using-context-api-in-react/)