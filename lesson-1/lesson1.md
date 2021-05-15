# **Solidity**

> **크립토좀비로 솔리디티 공부하기** [링크](https://cryptozombies.io)

## **1. 컨트랙트**

솔리디티 코드는 컨트렉트 안에 싸여 있다. `컨트랙트`는 이더리움 애플리케이션의 기본적인 구성 요소로, 모든 변수와 함수는 어느 한 컨트랙트에 속하게 되어있다.

비어 있는 HelloWorld 컨트랙트는 다음과 같다 :

```sol
contract HelloWorld {

}
```

### **Version Pragma**

모든 솔리디티 소스 코드는 "version pragma"로 시작해야 하는데, 이는 해당 코드가 이용해야 하는 솔리디티 버전을 선언하는 것이다. 이를 통해 이후에 새로운 컴파일러 버전이 나와도 기존 코드가 깨지지 않도록 예방하는 것이다.

컨트랙트 초기 뼈대는 다음과 같다. 새로운 프로젝트를 시작할 때마다 이 뼈대를 제일 먼저 작성해야 한다 :

```sol
pragma solidity ^0.4.19;

contract HelloWorld {

}
```

## **2. 상태 변수 & 정수**

`상태 변수`는 컨트랙트 저장소에 영구적으로 저장된다. 즉, 이더리움 블록체인에 기록되는 것이다. 데이터베이스에 데이터를 쓰는 것과 동일하다.

예시 :
```sol
contract Example {
    // 이 변수는 블록체인에 영구적으로 저장된다
    uint myUnsignedInteger = 100;
}
```

### **부호 없는 정수 : `uint`**

`utint` 자료형은 부호 없는 정수로, 값이 음수가 아니어야 한다. 부호 있는 정수를 위한 `int` 자료형도 있다.

> *참고 : 솔리디티에서 `uint`는 실제로 `uint256`, 즉 256비트 부호 없는 정수의 다른 표현이다. `uint8`, `uint16`, `uint32`등과 같이 `uint`를 더 적은 비트로 선언할 수도 있다.*

## **3. 수학 연산**

솔리디티의 연산은 대부분의 프로그래밍 언어의 수학 연산과 동일하다 :

* 덧셈 : `x + y`
* 뺄셈 : `x - y`
* 곱셈 : `x * y`
* 나눗셈 : `x / y`
* 나머지 : `x % y`

솔리디티는 `지수 연산`도 지원한다 :

```sol
uint x = 5 ** 2;    // 즉, 5^2 = 25
```

## **4. 구조체**

솔리디티는 구조체를 제공한다 :
```sol
struct Person {
    uint age;
    string name;
}
```

> *참고 : string 자료형은 임의의 길이를 가진 UTF-8 데이터를 위해 활용된다 :   
`string greeting = "Hello world!"`*

## **5. 배열**

솔리티티에는 정적 배열과 동적 배열 두 종류의 배열을 제공한다 :

```sol
uint[2] fixedArray;     // 2개의 원소를 담을 수 있는 고정 길이의 배열
string[5] stringArray;  // 또 다른 고정 배열로 5개의 스트링을 담을 수 있다
uint[] dynamicArray;    // 동적 배열은 고정된 크기가 없으며 계속 커질 수 있다
```

구조체의 배열을 생성할 수도 있다. 이전 챕터의 `Person` 구조체를 이용하면 :

```sol
Person[] people;        // 이는 동적 배열로, 원소를 계속 추가할 수 있다
```

상태 변수가 블록체인에 영구적으로 저장될 수 있다고 한 바 있다. 그러니 이처럼 구조체의 동적 배열을 생성하면 마치 데이터베이스처럼 컨트랙트에 구조화된 데이터를 저장하는 데 유용하다.

### **Public 배열**

`public`으로 배열을 선언할 수 있다. 솔리디티는 이런 배열을 위해 `getter` 메소드를 자동적으로 생성한다. 구문은 다음과 같다 :

```sol
Person[] public people;
```

그러면 다른 컨트랙트들이 이 배열을 읽을 수 있게 된다 (쓸 수는 없다). 이는 컨트랙트에 공개 데이터를 저장할 때 유용한 패턴이다.

## **6. 함수 선언**

솔리디티에서 함수 선언은 다음과 같이 한다 :

```sol
function eatHamburgers(string _name, uint _amout) {

}
```

위 함수는 `eatHamburgers`라는 함수로, `string`과 `uint` 2개의 인자를 전달받는다. 현재 함수의 내용은 비어있다.

> *참고 : 함수 인자명을 언더스코어(_)로 시작해서 전역 변수와 구별하는 것이 관례이다. (의무는 아님)*

이 함수를 다음과 같이 호출할 수 있다 :
```sol
eatHamburgers("vitalik", 100);
```

## **7. 구조체와 배열 활용하기**

새로운 구조체 생성하기 :

```sol
struct Person {
    uint age;
    string name;
}

Person[] public people;
```

새로운 Person를 생성하고 people 배열에 추가하는 방법 :

```sol
Person satoshi = Person(172, "Satoshi");
// 새로운 사람을 생성한다
people.push(satoshi);
// 이 사람을 배열에 추가한다
```

이 두 코드를 조합하여 한 줄로 표현할 수 있다 :

```sol
people.push(Person(16, "Vitalik"));
```

> *`arrary.push()`는 무언가를 배열의 끝에 추가해서 모든 원소가 순서를 유지하도록 한다.*

예시 :

```sol
uint[] numbers;
numbers.push(5);
numbers.push(10);
numbers.push(15);
// number 배열은 [5, 10, 15]과 같다
```

## **8. Private / Public 함수**

솔리디티에서 함수는 기본적으로 `public`으로 선언된다. 즉, 누구나 (혹은 다른 어느 컨트랙트가) 나의 컨트랙트의 함수를 호출하고 코드를 실행할 수 있다는 것이다.

확실히 이는 항상 바람직한 건 아닐 뿐더러, 나의 컨트랙트를 공격에 취약하게 만들 수 있다. 그러므로 기본적으로 함수를 `private`으로 선언하고, 공개할 함수만 `public`으로 선언하는 것이 좋다.

`private` 함수를 선언하는 방법 :

```sol
uint[] numbers;

function _addToArray(uint _number) private {
    numbers.push(_number);
}
```

`private`는 컨트랙트 내의 다른 함수들만이 이 함수를 호출하여 `numbers` 배열로 무언가를 추가할 수 있다는 것을 의미한다.

위의 예시에서 볼 수 있듯이 `private` 키워드는 함수명 다음에 적는다. 함수 인자명과 마찬가지로 `private` 함수명도 언더바(_)로 시작하는 것이 관례이다.

## **9. 함수 더 알아보기**

### **반환값**

함수에서 어떤 값을 반환 받으려면 다음과 같이 선언해야 한다 :

```sol
string greeting = "What's up dog";

function sayHello() public returns (string) {
    return greeting;
}
```

솔리디티에서 함수 선언은 반환값 종류를 포함한다. (이 경우는 `string`)

### **함수 제어자**

위에서 살펴 본 함수 `sayHello()`는 솔리디티에서 상태를 변화시키지 않는다. 즉, 어떤 값을 변경하거나 무언가를 쓰지 않는다.

이 경우에는 함수를 `view` 함수로 선언한다. 이는 함수가 데이터를 보기만 하고 변경시키지 않는 다는 뜻이다 :

```sol
function sayHello() public view returns (string) {
```

솔리디티는 `pure` 함수도 가지고 있는데, 이는 함수가 앱에서 어떤 데이터도 접근하지 않는 다는 것을 의미한다 :

```sol
function _multiply(uint a, uint b) private pure returns (uint) {
    return a * b;
}
```

이 함수는 앱에서 읽는 것도 하지 않고, 다만 반환값이 함수에 전달된 인자값에 따라서 달라진다. 그래서 이 경우에 함수를 `pure`로 선언하는 것이다.

> *참고 : 함수를 `pure`나 `view`로 언제 표시할지 헷갈릴 수 있다. 다행히 솔리디티 컴파일러는 어떤 제어자를 써야 하는지 경고 메시지를 통해 알려준다.*

## **10. Keccak256과 형 변환**

이더리움은 `SHA3`의 한 버전인 `keccak256`를 내장 해시 함수로 가지고 있다. 해시 함수는 기본적으로 입력 스트링을 랜덤 256비트 16진수로 매핑한다. 스트링에 약간의 변화라도 있으면 해시 값은 크게 달라진다.

해시 함수는 이더리움에서 여러 용도로 활용되지만, 여기서는 의사 난수 발생기 (pseudo-random number generator)로 이용하겠다.

아래 예시를 보면 입력값의 한 글자만 달라졌음에도 반환값은 완전히 달라짐을 알 수 있다 :
```sol
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256("aaaab");
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256("aaaac");
```

> *참고 : 블록체인에서 안전한 의사 난수 발생기는 매우 어려운 문제이다. 여기서 활용된 방법은 안전하지는 않지만 지금은 보안을 최우선순위에 두고 활용하는 것이 아니기 때문에 크게 신경쓰지 않아도 된다.*

### **형 변환**

자료형 간에 변환이 필요할 때가 있다 :

```sol
uint8 a = 5;
uint b = 6;
// a * b가 uint8이 아닌 uint를 반환하기 때문에 에러 메시지가 난다 :
uint8 c = a * b;
// b를 uint8으로 형 변환해서 코드가 제대로 작동하도록 해야 한다 :
uint8 c = a * uint8(b);
```

## **11. 이벤트**

`이벤트`는 나의 컨트랙트가 블록체인 상에서 나의 앱의 사용자 단에서 무언가 액션이 발생했을 때 의사소통하는 방법이다. 특정 이벤트가 일어나는지 "귀를 기울이고" 그 이벤트가 발생하면 행동을 취한다.

예시 :

```sol
// 이벤트를 선언한다
event IntegersAdded(uint x, uint y, utint result);

function add(uint _x, uint _y) public {
    uint result = _x + _y;
    // 이벤트를 실행하여 앱에게 add 함수가 실행되었음을 알린다
    IntegersAdded(_x, _y, result);
    return result;
}
```

그러면 나의 앱의 사용자 단은 해당 이벤트가 일어나는지 귀를 귀울인다.

자바스크립트로 이를 구현하면 다음과 같다 :

```javascript
YourContract.IntegersAdded(function(error, result) {
    // 결과와 관련된 행동을 취한다
})
```

## **12. 종합하기**

```sol
pragma solidity ^0.4.19;

contract ZombieFactory {

    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```

## **13. Web3.js**

이더리움은 `Web3.js`라고 하는 자바스크립트 라이브러리를 가지고 있다.

`Web3.js`가 구축된 컨트랙트와 어떤 방식으로 상호작용하는지에 대한 샘플 코드 :

```javascript
// 여기에 우리가 만든 컨트랙트에 접근하는 방법을 제시한다:
var abi = /* abi generated by the compiler */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory`는 우리 컨트랙트의 public 함수와 이벤트에 접근할 수 있다.

// 일종의 이벤트 리스너가 텍스트 입력값을 취한다:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  // 우리 컨트랙트의 `createRandomZombie`함수를 호출한다:
  ZombieFactory.createRandomZombie(name)
})

// `NewZombie` 이벤트가 발생하면 사용자 인터페이스를 업데이트한다
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  generateZombie(result.zombieId, result.name, result.dna)
})

// 좀비 DNA 값을 받아서 이미지를 업데이트한다
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // DNA 값이 16자리 수보다 작은 경우 앞 자리를 0으로 채운다
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // 첫 2자리는 머리의 타입을 결정한다. 머리 타입에는 7가지가 있다. 그래서 모듈로(%) 7 연산을 하여
    // 0에서 6 중 하나의 값을 얻고 여기에 1을 더해서 1에서 7까지의 숫자를 만든다. 
    // 이를 기초로 "head1.png"에서 "head7.png" 중 하나의 이미지를 불러온다:
    headChoice: dnaStr.substring(0, 2) % 7 + 1,
    // 두번째 2자리는 눈 모양을 결정한다. 눈 모양에는 11가지가 있다:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 셔츠 타입에는 6가지가 있다:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    // 마지막 6자리는 색깔을 결정하며, 360도(degree)까지 지원하는 CSS의 "filter: hue-rotate"를 이용하여 아래와 같이 업데이트된다:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
```