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

## **1. Msg.sender**

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