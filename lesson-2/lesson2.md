# **Cryptozombies[lesson2]**

> **크립토좀비로 솔리디티 공부하기** [링크](https://cryptozombies.io)

## **1. 매핑과 주소**

### **주소**

이더리움 블록체인은 은행 계좌와 같은 계정들로 이루어져 있다. 계정은 이더리움 블록체인 상의 통화인 `이더`의 잔액을 가진다. 은행 계좌에서 다른 계좌로 돈을 송금할 수 있듯이, 계정을 통해 다른 계정으로 이더를 주고 받을 수 있다.

각 계정은 은행 계좌 번호와 같은 `주소`를 가지고 있다. 주소는 특정 계정을 가리키는 고유 식별자로, 다음과 같이 표현된다 :

`0x0cE446255506E92DF41614C46F1d6df9Cc969183`
(크립토좀비 팀의 이더리움 주소)

주소는 특정 유저(혹은 스마트 컨트랙트)가 소유한다. 그러니까 주소를 좀비들에 대한 소유권을 나타내는 고유 ID로 활용할 수 있다. 유저가 이 앱을 통해 새로운 좀비를 생성하면 좀비를 생성하는 함수를 호출한 이더리움 주소에 그 좀비에 대한 소유권을 부여하는 것이다.

### **매핑**

lesson 1에서는 `구조체`와 `배열`을 살펴보았다. `매핑`은 솔리디티에서 구조화된 데이터를 저장하는 또다른 방법이다.

다음과 같이 매핑을 정의한다 :

```sol
// 금융 앱용으로, 유저의 계좌 잔액을 보유하는 uint를 저장한다 :
mapping (address => uint) public accountBalance;
// 혹은 userID로 유저 이름을 저장/검색하는 데 매핑을 쓸 수도 있다 :
mapping (uint => string) userIdToName;
```

매핑은 기본적으로 키-값(`key-value`) 저장소로, 데이터를 저장하고 검색하는 데 이용된다. 첫번째 예시에서 키는 `address`이고 값은 `uint`이다. 두번째 예시에서 키는 `uint`이고 값은 `string`이다.

## **2. Msg.sender**

솔리디티에는 모든 함수에서 이용 가능한 특정 전역 변수들이 있다. 그 중의 하나가 현재 함수를 호출한 사람(혹은 스마트 컨트랙트)의 주소를 가르키는 msg.sender이다.

> *참고 : 솔리디티에서 함수 실행은 항상 외부 호출자가 시작한다. 컨트랙트는 누군가가 컨트랙트의 함수를 호출할 때까지 블록체인 상에서 아무것도 안하고 있을 것이다. 그러니 항상 `msg.sender`가 있어야 한다.*

`msg.sender`를 이용하고 `mapping`을 업데이트하는 예시 :

```sol
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _mynumber) public {
    // msg.sender에 대해 _myNumber가 저장되도록 favoriteNumber 매핑을 업데이트한다
    favoriteNumber[msg.sender] = _myNumber;
    // 데이터를 저장하는 구문은 배열로 데이터를 저장할 때와 동일하다
}

function whatIsMyNumber() public view returns (uint) {
    // sender의 주소에 저장된 값을 불러온다
    // sender가 setMyNumber을 아직 호출하지 않았다면 반환값은 0이 될 것이다
    return favoriteNumber[msg.sender];
}
```

위의 예시에서는 누구나 `setMyNumber`을 호출하여 본인의 주소와 연결된 컨트랙트 내에 `uint`를 저장할 수 있다.

`msg.sender`를 활용하면 이더리움 블록체인의 보안성을 이용할 수 있게 된다. 즉, 누군가 다른 사람의 데이터를 변경하려면 해당 이더리움 주소와 관련된 개인키를 훔치는 것 밖에는 다른 방법이 없다는 것이다.

## **3. Require**

`requre`를 활용하면 특정 조건이 참이 아닐 때 함수가 에러 메시지를 발생하고 실행을 멈추게 된다 :

```sol
function sayHiToVitalik(string_name) public returns (string) {
    // _name이 "Vitalik"인지 비교하고 참이 아닐 경우 에러 메시지를 발생하고 함수를 벗어난다
    // (참고 : 솔리디티는 고유의 스트링 비교 기능을 가지고 있지 않기 때문에 스트링의 keccak256 해시값을 비교하여 스트링 값이 같은지 판단한다)
    require(keccak256(_name) == keccak256("Vitalik"));
    // 참이면 함수를 실행한다 :
    return "Hi";
}
```

`sayHiToVitalik("Vitalik")`로 이 함수를 실행하면 `Hi!`가 반환될 것이다. `Vitalik`이 아닌 다른 값으로 이 함수를 호출할 경우, 에러 메시지가 뜨고 함수가 실행되지 않을 것이다.

`require`는 함수를 실행하기 전에 참이어야 하는 특정 조건을 확인하는 데 있어 유용하다.

## **4. 상속**

코드가 늘어나서 긴 컨트랙트 하나를 만들기 보다는 코드를 잘 정리해서 여러 컨트랙트에 코드 로직을 나누는 것이 합리적일 때가 있다.

이를 보다 관리하기 쉽도록 하는 솔리디티 기능이 바로 컨트랙트 `상속`이다 :

```sol
contract Doge {
    function catchphrase() public returns (string) {
        return "So Wow CryptoDoge";
    }
}

contract BabyDoge is Doge {
    function anotherCatchphrase() public returns (string) {
        return "Such Moon BabyDoge";
    }
}
```

`BabyDoge` 컨트랙트는 `Doge` 컨트랙트를 상속한다. 즉, `BabyDoge` 컨트랙트를 컴파일해서 구축할 때, `BabyDoge` 컨트랙트가 `catchphrase()` 함수와 `anotherCatchphrase()` 함수에 모두 접근할 수 있다는 뜻이다. 
(`Doge` 컨트랙트에 다른 어떤 public 함수가 정의되어도 접근이 가능하다)

상속 개념은 `"고양이는 동물이다"`의 경우처럼 부분집합 클래스가 있을 때 논리적 상속을 위해 활용할 수 있다. 하지만 또한 로직을 다수의 클래스로 분할해서 단순히 코드를 정리할 때도 활용할 수 있다.

## **5. Import**

다수의 파일이 있고 어떤 파일을 다른 파일로 불러오고 싶을 때 솔리디티는 `import`라는 키워드를 이용한다 :

```sol
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

이 컨트랙트와 동일한 폴더에 (`./`) `someothercontract.sol`이라는 파일이 있을 때 이 파일을 컴파일러가 불러오게 된다.

## **6. Storage vs Memory**

솔리디티에는 변수를 저장할 수 있는 공간으로 `storage`와 `memory` 두가지가 있다.

`Storage`는 블록체인 상에 영구적으로 저장되는 변수이다. `Memory`는 임시적으로 저장되는 변수로, 컨트랙트 함수에 대한 외부 호출들이 일어나는 사이에 지워진다. 두 변수는 각각 컴퓨터 하드 디스크와 RAM과 같다.

대부분의 경우에 사용자가 이런 키워드들을 이용할 필요가 없다. 솔라디티가 알아서 처리해 주기 때문이다. 상태변수(함수 외부에 선언된 변수)는 초기 설정 상 `storage`로 선언되어 블록체인에 영구적으로 저장되는 반면, 함수 내에 선언된 변수는 `memory`로 자동 선언되어서 함수 호출이 종료되면 사라진다.

하지만 이 키워드들을 사용해야하는 때가 있다. 바로 함수 내의 `구조체`와 `배열`을 처리할 때다 :

```sol
contract SandwichFactory {
    struct Sandwitch {
        string name;
        string status;
    }

    Sandwich[] sandwiches;

    function eatSandwich(uint _index) public {
        // Sandwich mySandwich = sandwich[_index];

        // 꽤 간단해 보이나, 솔리디티는 여기서 storage나 memory를 명시적으로 선언해야 한다는 경고 메시지를 발생한다
        // 그러므로 storage 키워드를 활용하여 다음과 같이 선언해야 한다 :
        Sandwich storage mySandwich = sandwich[_index];
        // 이 경우, mySandwich는 저장된 sandwich[_index]를 가리키는 포인터이다
        mySandwich.status = "Eaten!";
        // 이 코드는 블록체인 상에서 'sandwiches[_index]`을 영구적으로 변경한다

        // 단순히 복사를 하고자 한다면 memory를 이용하면 된다 :
        Sandwich memory anotherSandwich = sandwiches[_index + 1];
        // 이 경우, anotherSandwich는 단순히 메모리에 데이터를 복사하는 것이 된다
        anotherSandwich.status = "Eaten!";
        // 이 코드는 임시변수인 anotherSandwich를 변경하는 것으로 sandwich[_index + 1]에는 아무런 영향을 끼치지 않는다
        // 그러나 다음과 같이 코드를 작성할 수 있다 :
        sandwiches[_index + 1] = anotherSandwich;
        // 이는 임시 변경한 내용을 블록체인 저장소에 저장하고자 하는 경우이다
    }
}
```

지금으로썬 명시적으로 `storage`나 `memory`를 선언할 필요가 있는 경우가 있다는 걸 이해하는 것만으로도 충분하다.

## **7. 함수 접근 제어자 더 알아보기**

### **Internal과 External**

`public`과 `private` 이외에도 솔리디티에는 `internal`과 `external`이라는 함수 접근 제어자가 있다.

`internal`은 함수가 정의된 컨트랙트를 상속하는 컨트랙트에서도 접근이 가능하다는 점을 제외하면 `private`함수와 동일하다.

`external`은 함수가 컨트랙트 바깥에서만 호출될 수 있고 컨트랙트 내의 다른 함수에 의해 호출될 수 없다는 점을 제외하면 `public`과 동일하다.

`internal`이나 `external`함수를 선언하는 건 `private`과 `public`함수를 선언하는 구문과 동일하다 :

```sol
contract Sandwich {
    uint private sandwichesEaten = 0;

    function eat() internal {
        sandwichesEaten++;
    }
}

contract BLT is Sandwich {
    uint private baconSandwichesEaten = 0;

    function eatWithBacon() public returns (string) {
        baconSandwichesEaten++;
        // eat 함수가 internal로 선언되었기 때문에 여기서 호출이 가능하다
        eat();
    }
}
```

## **8. 다른 컨트랙트와 상호작용하기**

블록체인 상에 있으면서 우리가 소유하지 않은 컨트랙트와 우리 컨트랙트가 상호작용을 하려면 우선 `인터페이스`를 정의해야 한다.

예를들어 다음과 같은 블록체인 컨트랙트가 있다고 해보자 :

```sol
contract LuckyNumber {
    mapping(address => uint) numbers;

    function setNum(uint _num) public {
        numbers[msg.sender] = _num;
    }

    function getNum(address _myAddress) public view returns (uint) {
        return numbers[_myAddress];
    }
}
```

이 컨트랙트는 아무나 자신의 행운의 수를 저장할 수 있는 간단한 컨트랙트이고, 각자의 이더리움 주소와 연관이 있을 것이다. 이 주소를 이용해서 누구나 그 사람의 행운의 수를 찾아볼 수 있다.

이제 `getNum`함수를 이용하여 이 컨트랙트에 있는 데이터를 읽고자 하는 `external`함수가 있다고 해보자.

먼저, `LuckyNumber` 컨트랙트의 `인터페이스`를 정의할 필요가 있다 :

```sol
contract NumberInterface {
    function getNum(address _myAddress) public view returns (uint);
}
```

약간 다르지만, 인터페이스를 정의하는 것이 컨트랙트를 정의하는 것과 유사하다는 걸 참고하자. 먼저, 다른 컨트랙트와 상호작용하고자 하는 함수만을 선언할 뿐(이 경우, `getNum`이 바로 그러한 함수이다) 다른 함수나 상태 변수를 언급하지 않는다.

다음으로, 함수 몸체를 정의하지 않는다. 중괄호 `{`, `}`를 쓰지 않고 함수 선언을 세미콜론(`;`)으로 간단하게 끝낸다.

그러니 인터페이스는 컨트랙트 뼈대처럼 보인다고 할 수 있다. 컴파일러도 그렇게 인터페이스를 인식한다.

`dapp`코드에 이런 인터페이스를 포함하면 컨트랙트는 다른 컨트랙트에 정의된 함수의 특성, 호출 방법, 예상되는 응답 내용에 대해 알 수 있게 된다.

> *참고 : 솔라디티에서는 함수가 하나 이상의 값을 반환할 수 있다*