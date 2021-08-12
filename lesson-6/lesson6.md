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