# **Cryptozombies[lesson6]**

> **크립토좀비로 솔리디티 공부하기** [링크](https://cryptozombies.io)

## **1. `Web3.js` 소개**

`lesson5`에서 좀비 `DApp`이 완성되었고 이제는 사용자들이 상호작용 할 수 있는 기본적인 웹 페이지를 만들 것이다. 이를 만들기 위해, 이더리움 재단에서 만든 자바스크립트 라이브러리인 `Web3.js`를 사용할 것이다.

### `Web3.js`란?

이더리움 네트워크는 노드로 구성되어 있고, 각 노드는 블록체인의 복사본을 가지고 있다. 누구든 스마트 컨트랙트의 함수를 실행하고자 한다면, 이 노드들 중 하나의 질의를 보내 아래 내용을 전달해야 한다 :

1. 스마트 컨트랙트 주소
2. 실행하고자 하는 함수, 그리고
3. 그 함수에 전달하고자 하는 변수들

이더리움 노드들은 `JSON-RPC`라고 불리는 언어로만 소통할 수 있고, 이는 사람이 읽기는 불편하다. 컨트랙트의 함수를 실행하고 싶다고 질의를 보내는 것은 다음과 같이 생겼다 :

```json
{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}
```

다행히도 `Web3.js`는 이런 골치 아픈 질의를 몰라도 된다. 편리하고 쉽게 읽을 수 있는 자바스크립트 인터페이스로 상호작용을 하면 되는 것이다.

위의 질의문을 작성할 필요 없이, 코드에서 함수를 호출하는 것은 다음과 같을 것이다 :

```js
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔") .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```

프로젝트의 작업 흐름에 맞춰, 프로젝트에서 가장 많이 사용하는 패키지 도구를 써서 `Web3.js`를 추가할 수 있다 :

```html
<script language="javascript" type="text/javascript" src="web3.min.js"></script>
```

## **2. Web3 프로바이더(Provider)**

우리의 프로젝트에 `Web3.js`를 넣었으니 이제 이를 초기화하고 블록체인과 대화를 해보도록 하자.

우리가 처음 필요로 하는 것은 `Web3 프로바이더(Provider)`이다.

이더리움은 똑같은 데이터의 복사본을 공유하는 `노드`들로 구성되어 있다. `Web3.js`에서 `Web3 프로바이더`를 설정하는 것은 우리 코드에 읽기와 쓰기를 처리할 때 어떤 노드와 통신을 해야 하는지 설정하는 것이다. 이는 전통적인 웹 앱에서 `API`호출을 위해 원격 웹 서버의 `URL`을 설정하는 것과 같다.

나는 나만의 이더리움 노드를 프로바이더로 운영할 수도 있다. 하지만 내가 편리하게 쓸 수 있는 제 3자 서비스가 있다. 내 `DApp`의 사용자들을 위해 나만의 이더리움 노드를 운영할 필요가 없도록 하기 위해 사용할 수 있는 `Infura`라는 서비스가 있다.

### **Infura**

`Infura`는 빠른 읽기를 위한 캐시 계층을 포함하는 다수의 이더리움 노드를 운영하는 서비스이다. 접근을 위한 `API`를 무료로 사용할 수 있다. `Infura`를 프로바이더로 사용하면, 나만의 이더리움을 설치하고 계속 유지할 필요 없이 이더리움 블록체인과 메세지를 확실히 주고받을 수 있다.

다음과 같이 `Web3`에 나의 `Web3 프로바이더`로 `Infura`를 쓰도록 설정할 수 있다 :

```js
var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
```

하지만 많은 사용자들이 우리의 `DApp`을 사용할 것이기에, 그리고 이 사용자들은 단순히 읽기만 하는 게 아니라 블록체인에 뭔가 쓰기도 할 것이기에 우리는 이 사용자들이 그들의 개인 키로 트랜잭션에 서명을 할 수 있도록 해야 할 것이다.

> 참고 : 이더리움(그리고 일반적인 블록체인)은 트랙잭션에 전자 서명을 하기 위해 공개/개인 키 쌍을 사용한다. 말하자면 전자 서명을 위해 엄청나게 안전한 비밀번호 같은 것이다. 이런 방식으로 내가 만약 블록체인에서 어떤 데이터를 변경하면, 나의 공개 키를 통해 내가 거기 서명을 한 사람이라고 `증명`할 수 있다. 하지만 아무도 내 개인 키를 모르기 때문에 내 트랜잭션을 누구도 위조할 수 없다.

이런 암호학은 복잡하다. 그러니 우리의 앱 프론트엔드에서 사용자들의 개인 키를 관리하려 하는 것은 좋은 생각이 아니다.

그리고 이를 대신 처리해주는 서비스가 이미 있다. 이중 가장 유명한 것은 `메타마스크(Metamask)`이다.

### **메타마스크(Metamask)**

`메타마스크`는 사용자들이 이더리움 계정과 개인 키를 안전하게 관리할 수 있게 해주는 크롬과 파이어폭스의 브라우저 확장 프로그램이다. 그리고 해당 계정들을 써서 `Web3.js`를 사용하는 웹사이트들과 상호작용을 할 수 있도록 해준다. (메타마스크를 설치하면 브라우저에서 `Web3`가 작동하고, 이더리움 블록체인과 통신하는 어떤 웹사이트와도 상호작용을 할 수 있다.)

그리고 누군가가 개발자로서, 사용자들이 웹 브라우저를 써서 웹 사이트를 통해 `DApp`과 상호작용을 하길 원한다면(크립토좀비와 같이), 분명 그는 자기의 `DApp`을 메타마스크와 호환할 수 있게 하고 싶을 것이다.

> 참고 : 메타마스크는 내부적으로 `Infura`의 서버를 `Web3 프로바이더`로 사용한다. 위의 내용처럼 말이다. 하지만 사용자들에게 그들만의 `Web3 프로바이더`를 선택할 수 있는 옵션을 주기도 한다. 즉 메타마스크의  `Web3 프로바이더`를 사용한다면, 사용자에게 선택권을 주는 것이기도 하면서 앱에서 걱정할 거리를 하나 줄일 수 있게 되는 것이다.

### **메타마스크의 Web3 프로바이더 사용하기**

메타마스크는 `web3`라는 전역 자바스크립트 객체를 통해 브라우저에게 `Web3 프로바이더`를 주입한다. 그러니 내 앱에서 `web3`가 존재하는지 확인하고, 만약 존재한다면 `web3.currentProvider`를 프로바이더로서 사용하면 된다.

여기 메타마스크에서 제공하는 템플릿 코드가 있다. 사용자가 메타마스크를 설치했느지 확인하고 설치가 안 된 경우 앱을 사용하려면 메타마스크를 설치해야 한다고 알려준다 :

```js
window.addEventListener('load', function() {
	// Web3가 브라우저에 주입되었느지 확인(Mist/MetaMask)
	if (typeof web3 !== 'undefined') {
		// Mist/MetaMask의 프로바이더 사용
		web3js = new Web3(web3.currentProvider);
	} else {
		// 사용자가 Metamask를 설치하지 않은 경우에 대해 처리
		// 사용자들에게 Metamask를 설치하라는 등의 메세지를 보여줄 것
	}
	// 이제 앱을 시작하고 web3에 자유롭게 접근할 수 있다 :
	startApp()
})
```

나는 내가 만드는 모든 앱에서 이 예제 코드를 사용할 수 있다. 사용자들이 내 앱을 사용하려면 메타마스크를 사용하도록 하기 위해서 말이다.

> 참고 : 메타마스크 말고도 사용자들이 쓸 수 있는 다른 개인 키 관리 프로그램도 있다. `미스트(Mist)` 웹 브라우저 같은 것들이다. 하지만 그것들도 모두 `web3`변수를 주입하는 동일한 형태를 사용한다. 그러니 사용자들이 다른 프로그램을 쓰더라도 여기서 설명하는 방식으로 사용자의 `Web3 프로바이더`를 인식할 수 있을 것이다.

## **3. 컨트랙트와 대화하기**

`Web3.js`가 나의 스마트 컨트랙트와 통신을 하기 위해서는 2가지를 필요로 할 것이다 : `컨트랙트의 주소`와 `ABI`이다.

### **컨트랙트 주소**

스마트 컨트랙트를 모두 작성한 다음, 컴파일한 후 이더리움에 배포할 것이다. 배포는 다음 레슨에서 다룰 것이다. 이는 코드를 작성하는 것과는 꽤나 다른 절차이기 때문에 순서를 바꿔 `Web3.js`를 먼저 다루기로 한 것이다.

컨트랙트를 배포한 후, 해당 컨트랙트는 영원히 존재하는 이더리움 상에서 고정된 주소를 얻을 것이다. `lesson2`를 상기해보면, 이더리움 메인넷에서 크립토키티의 주소는 `0x06012c8cf97BEaD5deAe237070F9587f8E7A266d` 였다.

나의 스마트 컨트랙트와 통신을 하기 위해 배포 후 이 주소를 복사해야 할 것이다.

### **컨트랙트 ABI**

컨트랙트와의 통신을 위해 `Web3.js`에서 필요로 하는 다른 하나는 바로 컨트랙트의 `ABI`이다.

`ABI`는 `Application Binary Interface`의 줄임말이다. 기본적으로 `JSON` 형태로 나의 컨트랙트 메소드를 표현하는 것이다. 나의 컨트랙트가 이해할 수 있도록 하려면 `Web3.js`가 어떤 형태로 함수 호출을 해야 하는지 알려주는 것이다.

이더리움에 배포하기 위해 컨트랙트를 컴파일할 때(lesson7에서 다룰 내용), 솔리디티 컴파일러가 나에게 `ABI`를 줄 것이다. 그러니 컨트랙트 주소와 함께 이를 복사하여 저장해야 한다.

### **`Web3.js` 컨트랙트 인스턴스화하기**

컨트랙트의 주소와 `ABI`를 얻고 나면, 다음과 같이 `Web3`에서 인스턴스화 할 수 있다 :

```js
// myContract 인스턴스화
var myContract = new web3js.eth.Contract(myABI, myContractAddress);
```

## **4. 컨트랙트 함수 호출하기**

컨트랙트의 설정이 끝났으니 이제 우리는 `Web3.js`로 컨트랙트와 통신할 수 있다. `Web3.js`는 컨트랙트의 함수를 호출하기 위해 사용할 수 있는 두 개의  메소드를 가지고 있다. 바로 `call`과 `send`이다.

### **Call**

`call`은 `view`와 `pure` 함수를 위해 사용한다. 로컬 노드에서만 실행하고, 블록체인 트랜잭션을 만들지 않는다.

> 복습 : `view`와 `pure` 함수는 읽기 전용이고 블록체인에서 상태를 변경하지 않는다. 가스를 전혀 소모하지 않고, 메타마스크에서 트랜잭션에 서명하라고 사용자에게 창을 띄우지도 않는다.

`Web3.js`를 사용하여, 다음과 같이 `123`을 매개변수로 `myMethod`라는 이름의 함수를 `call`할 수 있다 :

```js
myContract.methods.myMethod(123).call()
```

### **Send**

`send`는 트랜잭션을 만들고 블록체인 상의 데이터를 변경한다. `view`와 `pure`가 아닌 모든 함수에 대해 `send`를 사용해야 하는 것이다.

> 참고 : 트랜잭션을 `send`하는 것은 사용자에게 가스를 지불하도록 하고, 메타마스크에서 트랜잭션에 서명하고 창을 띄울 것이다. `Web3 프로바이더`로 메타마스크를 사용할 때, `send()`를 호출하면 자동으로 이 모든 것이 이루어지고, 우리의 코드에 어떤 특별한 것도 추가할 필요가 없다.

`Web3.js`를 사용하여, 다음과 같이 `123`을 매개변수로 `myMethod`라는 이름의 함수를 호출하는 트랜잭션을 `send`할 수 있다 :

```js
myContract.method.myMethod(123).send()
```

구문은 `call()`과 거의 똑같다.

### **좀비 데이터 받기**

이제 컨트랙트에서 데이터에 접근하기 위해 `call`을 사용하는 에졔를 살펴볼 것이다.

우리가 좀비 배역을 `public`으로 만들었던 것을 기억하자 :

```sol
Zombie[] public zombies;
```

솔리디티에서 `public`으로 변수를 선언하면 자동으로 같은 이름의 퍼블릭 "getter" 함수를 만들어 낸다. 그러니 `ID 15`인 좀비를 찾길 원한다면, 변수를 함수인 것처럼 호출할 수 있다 : `zombies(15)`

여기에 우리의 프론트엔드에서 좀비 `ID`를 받아 해당 좀비에 대해 컨트랙트에 질의를 보내고, 결과를 반환하는 자바스크립트 함수르 작성하는 방법이 있다 :

> 참고 : 이번 레슨에서 우리가 사용하는 모든 코드 예제들은 콜백 대신 `Promise`를 사용하는 `Web3.js 1.0 버전`을 사용하고 있다. 만약 다른 튜토리얼에서 코드를 복사해온다면 같은 버전을 사용하고 있는지 확인해봐야 한다.

```js
function getZombieDetails(id) {
	return cryptoZombies.methods.zombies(id).call()
}

// 함수를 호출하고 결과를 가지고 무언가를 처리 :
getZombieDetails(15)
.then(function(result) {
	console.log("Zombie 15: " + JSON.stringify(result));
});
```

여기서 일어나고 있는 것들을 알아보도록 하자.

`cryptoZombies.methods.zombies(id).call()`은 `Web3 프로바이더`와 통신하여 우리 컨트랙트의 `Zombie[] public zombies`에서 인덱스가 `id`인 좀비를 반환하도록 할 것이다.

이는 외구 서버로 `API`호출을 하는 것처럼 `비동기`적으로 일어난다는 것을 알아두자. 즉 `Web3`는 여기서 `Promise`를 반환한다.

`Promise`가 만들어지면(이는 `Web3 프로바이더`로부터 응답을 받았다는 것을 의미) 우리 예제 코드는 `then`문장을 실행하고, 여기서 `result`를 콘솔에 로그로 기록한다.

`result`는 다음과 같이 생긴 자바스크립트 객체가 될 것이다 :

```js
{
	"name": "H4XF13LD MORRIS'S COOLER OLDER BROTHER",
	"dna": "1337133713371337",
	"level": "9999",
	"readyTime": "1522498671",
	"winCount": "999999999",
	"lossCount": "0" // Obviously.
}
```

이후 이 겍체를 해석하기 위한 프론트엔드 로직을 만들어 의미 있는 방향으로 이 객체를 프론트엔드에 표시할 것이다.

## **5. 메타마스크 & 계정**

이제 각 요소들을 하나로 합쳐나가보도록 하겠다. 우리 앱의 홈페이지에 사용자의 전체 좀비 군대를 보여주고 싶다고 가정해보자.

현재 사용자가 가지고 있는 모든 좀비들의 `ID`를 찾기 위해 우리는 분명 첫 번째로 `getZombiesByOwner(owner)`함수를 사용해야 할 것이다.

하지만 우리의 솔리디티 컨트랙트는 `owner`에 솔리디티 `address`를 보내도록 되어 있다. 그렇다면 우리가 어떻게 우리 앱을 사용하는 사용자의 주소를 알 수 있겠는가?

### **메타마스크에서 사용자 계정 가져오기**

메타마스크는 확장 프로그램 안에서 사용자들이 다수의 계정을 관리할 수 있도록 해준다.

우리는 주입되어 있는 `web3` 변수에 현재 활성화된 계정이 무엇인지 다음과 같이 확인할 수 있다 :

```js
var userAccount = web3.eth.accounts[0]
```

사용자가 언제든지 메타마스크에서 활성화된 계정을 바꿀 수 있기 때문에, 우리 앱은 이 변수의 값이 바뀌었는지 확인하기 위해 계속 감시를 하고 값이 바뀌면 그에 따라 `UI`를 업데이트해야 할 것이다. 예를 들어, 사용자들의 홈페이지에서 그들의 좀비 군대를 표시하고 싶다면, 메타마스크에서 계정을 바꾸었을 때 바뀐 계정에 대한 좀비 군대를 보여주기 위해 페이지를 업데이트해야 할 것이다.

이를 위해 다음과 같이 `setInterval`을 쓸 수 있다 :

```js
var accountInterval = setInterval(function() {
	// 계정이 바뀌었는지 확인
	if (web3.eth.accounts[0] !== userAccount) {
		userAccount = web3.eth.accounts[0];
		// 새 계정에 대한 UI로 업데이트하기 위한 함수 호출
		updateInterface();
	}
}, 100);
```

여기서는 `userAccount`가 여전히 `web3.eth.accounts[0]`과 같은지 위해 100밀리초마다 확인하고 있다. (즉 사용자가 해당 계정을 활성화해 놓았는지 확인하는 것이다) 그렇지 않다면, `userAccount`에 현재 활성화된 계정을 다시 할당하고, 화면을 업데이트하기 위한 함수를 호출한다.

## **6. 좀비 군대 보여주기**

컨트랙트로부터 받은 데이터를 실제로 보여줄 수 없다면 지금까지 한 것은 아직 완전하다고 할 수 없다.

`React`나 `Vue.js` 같은 편리한 프론트엔드 프레임워크들이 있지만 여기서 이 것들을 다루기에는 너무 광범위하다.

그래서 `CryptoZombies.io`의 초점을 이더리움과 스마트 컨트랙트에 맞추기 위해, 우리는 `jQuery`를 이용한 간단한 예제를 통해 어떻게 스마트 컨트랙트에서 전달받은 데이터를 파싱하고 표현할 수 있을지 살펴볼 것이다.

### **좀비 데이터 보여주기 - 간단한 예제**

현재 작업 중인 `html` 문서에 빈 `<div id="zombies"></div>`와 빈 `displayZombies`함수를 추가했다.

지난 챕터에서 `startApp()` 내에서 `getZombiesByOwner`의 호출 결과를 써서 호출한 `displayZombies`를 떠올려 보자. 그 함수의 결과로 아래와 같이 생긴 좀비 `ID` 배열을 전달받을 수 있을 것이다 :

```js
[0, 13, 47]
```

따라서 `displayZombies`함수는 다음과 같은 것을 할 것이다 :

1. 먼저 이미 무언가가 `#zombies` `div`의 안에 들어 있아면 이 `div`의 내용을 비운다. (이렇게 함녀 사용자가 그들의 활성화된 `MetaMask` 계정을 변경하면 새로운 좀비 군대를 로딩하기 전에 기존의 것을 삭제할 것이다.)

2. 반복을 통해 각 `id`마다 `getZombieDetail(id)`를 호출해서 우리의 스마트 컨트랙트에서 좀비에 대한 모든 정보를 찾는다.

3. 화면에 표시하기 위해 `HTML` 템플릿에 좀비에 대한 정보를 집어넣고, 해당 템플릿을 `#zombies` `div`에 붙여넣는다.

여기서도 우린 기본 템플릿 엔진이 없는 `jQuery`를 이용하기 때문에, 보기 싫을 수 있지만, 다음의 간단한 예제처럼 각 좀비에 대한 정보를 출력할 수 있다 :

```html
// 우리 컨트랙트에서 좀비 상세 정보를 찾아, `zombie` 객체 변환
getZombieDetails(id)
.then(function(zombie) {
	// HTML에 변수를 넣기 위해 ES6의 "template literal" 사용
	// 각각을 #zombies div에 붙여넣기
	$("#zombies").append(`<div class="zombie">
		<ul>
			<li>Name: ${zombie.name}</li>
			<li>DNA: ${zombie.dna}</li>
			<li>Level: ${zombie.level}</li>
			<li>Wins: ${zombie.winCount}</li>
			<li>Losses: ${zombie.lossCount}</li>
			<li>Ready Time: ${zombie.readyTime}</li>
		</ul>
	</div>`);
});
```

## **좀비 스프라이트는 어떻게 표현하나요?**

위 예제에서는 DNA를 문자열로 간단히 표현해 보았다. 이것을 이미지로 바꿔서 좀비를 표현하려면 어떻게 해야될까?

우린 DNA 문자열을 부분 문자열로 나누고, 모든 2자리 숫자를 이미지에 대응시켜 아래와 같이 이 작업을 처리 했었다 :

```js
// 좀비의 머리를 표현하는 1-7의 정수 얻기
var head = parseInt(zombie.dna.substring(0, 2)) % 7 + 1

// 순차적인 파일 이름으로 7개의 머리 이미지를 가지고 있다 :
var headSrc = "../assets/zombieparts/head-" + head + ".png"
```

각 컴포넌트는 `CSS`의 절대 좌표 포지셔닝을 이용해 다른 이미지 위에 위치할 것이다.


## **7. 트랜잭션 보내기**

이제 우리의 `UI`는 사용자의 메타마스크 계정을 감지하고, 자동으로 좀비 군대를 홈페이지에 표현할 것이다.

이제 `send`함수를 이용해서 스마트 컨트랙트의 데이터를 변경하는 방법을 살펴보자.

이 함수에는 `call`함수와는 꽤 다른 부분이 있다 :

1. 트랜잭션을 전송(`send`)하려면 함수를 호출한 사람의 `from`주소가 필요하다. (솔리디티 코드에서는 `msg.sender`가 될 것이다.) 이는 우리 `DApp`의 사용자가 되아야 할 것이니, 메타마스크가 나타나 그들에게 서명을 하도록 할 것이다.

2. 트랜잭션 전송(`send`)은 가스를 소모한다.

3. 사용자가 트랜잭션 전송을 하고 난 후 실제로 블록체인에 적용될 때까지는 상당한 지연이 발생할 것이다. 트랜잭션이 블록에 포함될 때까지 기다려야 하는데, 이더리움의 평균 블록 시간이 15초이기 때문이다. 만약 이더리움에 보류 중인 거래가 많거나 사용자가 가스 가격을 지나치게 낮게 보낼 경우, 우리 트랜잭션이 블록에 포함되길 기다려야 하고 ,이는 몇 분씩 걸릴 수 있다.

	그러니 이 코드의 비동기적 특성을 다루기 위한 로직이 필요하게 될 것이다.

### **좀비 만들기**

이제 사용자가 호출할 우리 컨트랙트 내의 첫번째 함수를 예제로 살펴보자 :

```sol
function createRandomZombie(string _name) public {
	require(ownerZombieCount[msg.sender] == 0);
	uint randDna = _generateRandomDna(_name);
	randDna = randDna - randDna % 100;
	_createZombie(_name, randDna);
}
```

메타마스크를 사용해 `Web3.js`에서 위 함수를 호출하는 방법의 예제 :

```js
function createRandomZombie(name) {
	// 시간이 꽤 걸릴 수 있으니, 트랜잭션이 보내졌다는 것을 유저가 알 수 있도록 UI를 업데이트해야 함
	$("#txStatus").text("Creating new zombie on the blockchain. This may take a while...");
	// 우리 컨트랙트에 전송하기:
	return CryptoZombies.methods.createRandomZombie(name)
	.send({ from: userAccount })
	.on("receipt", function(receipt) {
		$("#txStatus").text("Successfully created " + name + "!");
		// 블록체인에 트랜잭션이 반영되었으며, UI를 다시 그려야 함
		getZombiesByOwner(userAccount).then(displayZombies);
	})
	.on("error", function(error) {
		// 사용자들에게 트랜잭션이 실패했음을 알려주기 위한 처리
		$("#txStatus").text(error);
	});
}
```

위 함수는 우리의 `Web3 프로바이더`에게 트랜잭션을 전송(`send`)하고, 몇 가지 이벤트 리스너들을 연결한다 :

* `receipt`는 트랜잭션이 이더리움의 블록에 포함될 때, 즉 좀비가 생성되고 우리의 컨트랙트에 저장되었을 때 발생하게 된다.
* `error`는 트랜잭션이 블록에 포함되지 못했을 때, 예를 들어 사용자가 충분한 가스를 전송하지 않았을 때 발생하게 된다. 우리는 우리의 `UI`를 통해 사용자에게 트랜잭션이 전송되지 않았음을 알리고, 다시 시도할 수 있도록 할 것이다.

> 참고 : `send`를 호출할 때 `gas`와 `gasPrice`를 선택적으로 지정할 수 있다. `.send({ from: userAccount, gas: 3000000 })`와 같이 말이다. 만약 지정하지 않는다면, 메타마스크는 사용자가 이 값들을 선택할 수 있도록 할 것이다.

## **8. Payable 함수 호출하기**

이제 `Web3.js`에서 특별한 처리가 필요한 다른 종류의 함수를 살펴볼 것이다. 바로 `payable`함수이다.

### **레벨업!**

`ZombieHelper`를 다시 생각해보면, 우린 사용자가 레벨업할 수 있는 곳에 `payable`함수를 추가했었다 :

```sol
function levelUp(uint _zombieId) external payable {
	require(msg.value == levelUpFee);
	zombies[_zombieId].level++;
}
```

함수를 이용해 이더를 보내는 법은 간단하지만, 이더가 아니라 `wei`로 얼마를 보낼지 정해야하는 제한이 있다.

### **Wei란?**

`wei`는 이더의 가장 작은 하위 단위이다 - 하나의 이더는 10^18개의 `wie`이다.

이건 세기에는 너무 많다. 하지만 운이 좋게도 `Web3.js`에는 이러한 작업을 해주는 변환 유틸리티가 있다.

```js
// 이렇게 하면 1 ETH를 Wei로 바꿀 것이다
web3js.utils.toWei("1");
```

우리 `DApp`에서, 우리는 `levelUpFee = 0.001 ether`로 설정했다. 그러니 `levelUp`함수를 호출할 때, 아래의 코드를 써서 사용자가 `0.001`이더를 보내게 할 수 있다 :

```js
CryptoZombies.methods.levelUp(zombieId)
.send({ from: userAccount, value: web3js.utils.toWei("0.001") })
```

## **9. 이벤트(Event) 구독하기**

`Web3.js`를 통해 컨트랙트와 상호작용 하는 것은 꽤 간단하다. 한번 환경을 구축하고 나면, 함수를 호출하고 트랜잭션을 전송하는 것은 일반적인 웹 `API`와 크게 다르지 않다.

여기 우리가 다루고자 하는 것이 하나 더 있다. 바로 컨트랙트에서 이벤트를 구독하는 것이다.

### **새로운 좀비 수신하기**

`zombiefactory.sol`을 다시 생각해보면, 새로운 좀비가 생성될 때마다 매번 호출되던 `NewZombie`라는 이벤트가 있었다 :

```sol
event NewZombie(uint zombieId, string name, uint dna);
```

`Web3.js`에서 이벤트를 구독하여 해당 이벤트가 발생할 때마다 `Web3 프로바이더`가 나의 코드 내의 어떠한 로직을 실행시키도록 할 수 있다 :

```js
cryptoZombies.events.NewZombie()
.on("data", function(event) {
	let zombie = event.returnValues;
	// event.returnValue 객체에서 이 이벤트의 세 가지 반환 값에 접근할 수 있다 :
	console.log("새로운 좀비가 태어났습니다!", zombie.zombieId, zombie.name, zombie.dna);
}).on("error", console.error);
```

이건 `DApp`에서 어떤 좀비가 생성되든지 항상 알림을 보낼 거라는 걸 명심하자. 현재 사용자의 좀비만이 아니라는 것이다. 현재 사용자가 만든 것에 대해서만 알림을 보내고 싶다면 어떻게 해야할까?

### **indexed 사용하기**

이벤트를 필터링하고 현재 사용자와 연관된 변경만을 수신하기 위해, 우리의 `ERC721`을 구현할 때 `Transfer`이벤트에서 했던 것처럼 우리의 솔리디티 컨트랙트에 `indexed`키워드를 사용해야 한다.

```sol
event Transfer(address indexed _from, address indexed, _to, uint256 _tokenId);
```

이 경우, `_from`과 `_to`가 `indexed` 되어 있기 때문에, 우리 프론트엔드의 이벤트 리스너에서 이들을 필터링할 수 있다 :

```js
// filter를 사용해 _to가 userAccount와 같을 때만 코드를 실행
cryptoZombies.events.Transfer({ filter: {_to: userAccount } })
.on("data", function(evnet) {
	let data = event.returnValues;
	// 현재 사용자가 방금 좀비를 받았다
	// 해당 좀비를 보여줄 수 있도록 UI를 업데이트할 수 있도록 여기에 추가
}).on("error", console.error);
```

`event`와 `indexed`영역을 사용하는 것은 나의 컨트랙트에서 변화를 감지하고 프론트엔드에 반영할 수 있게 하는 유용한 방법이다.

### **지난 이벤트에 대해 질의하기**

우린 `getPastEvents`를 이용해 지난 이벤트들에 대해 질의를 하고, `fromBlock`과 `toBlock` 필터들을 이용해 이벤트 로그에 대한 시간 범위를 솔리디티에 전달할 수 있다. (여기서 block은 이더리움 블록 번호를 나타낸다.)

```js
cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
.then(function(events) {
	// events는 우리가 위에서 했던 것처럼 반복 접근할 event 객체들의 배열이다.
	// 이 코드는 생성된 모든 좀비의 목록을 우리가 받을 수 있게 할 것이다.
});
```

위 메소드를 사용해서 시작 시간부터의 이벤트 로그들에 대해 질의를 할 수 있기 때문에, 이를 통해 흥미로운 사용 예시를 만들 수 있네: 이벤트를 저렴한 형태의 storage로 사용하는 것이다.

다시 생각해보면, 데이터를 블록체인에 기록하는 것은 솔리디티에서 가장 비싼 비용을 지불하는 작업 중 하나였다. 하지만 이벤트를 이용하는 것은 가스 측면에서 훨씬 더 저렴하다.

여기서 단점이 되는 부분은 스마트 컨트랙트 자체 안에서는 이벤트를 읽을 수 없다는 것이다. 하지만 히스토리로 블록체인에 기록하여 앱의 프론트엔드에서 읽기를 원하는 데이터가 있다면, 이는 새겨놓아야 할 중요한 사용 예시이다.

예를 들어, 우린 이것을 좀비 전투의 히스토리 기록용으로 사용할 수 있다. 좀비가 다른 좀비를 공격할 때마다, 그리고 누군가 이길 때마다 우린 이벤트를 생성할 수 있다. 스마트 컨트랙트는 추후 결과를 계산할 때 이 데이터가 필요하지 않지만, 사용자들이 앱의 프론트엔드에서 찾아볼 수 있는 유용한 데이터이다.
