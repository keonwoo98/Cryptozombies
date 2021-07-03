# **Cryptozombies[lesson3]**

> **크립토좀비로 솔리디티 공부하기** [링크](https://cryptozombies.io)

## **1. 컨트랙트의 불변성**

지금까지는 솔리디티는 자바스크립트 같은 다른 언어와 꽤 비슷해보였을 것이다. 하지만 이더리움 `DApp`에는 일반적인 애플리케이션과는 다른 여러가지 특성이 있다.

첫째로, 이더리움에 컨트랙트를 배포하고 나면, 컨트랙트는 `변하지 않는다.(Immutable)` 다시 말하자면 컨트랙트를 수정하거나 업데이트할 수 없다는 것이다.

컨트랙트로 배포한 최초의 코드는 항상, 블록체인에 영구적으로 존재한다. 이것이 바로 솔리디티에 있어서 보안이 굉장히 큰 이슈인 이유이다. 만약 내가 배포한 컨트랙트 코드에 결점이 있어도 그것을 이후에 고칠 수 있는 방법이 전혀 없다. 사용자들에게 결점을 보안한 다른 스마트 컨트랙트 주소를 쓰라고 말하고 다니라고 할 수 밖에 없을 것이다.

그러나 이것 또한 스마트 컨트랙트의 한 특징이다. 코드는 곧 법인 것이다. 내가 어떤 스마트 컨트랙트의 코드를 읽고 검증을 했다면, 내가 함수를 호출할 때마다, 코드에 쓰여진 그대로 함수가 실행될 것이라고 확신할 수 있다. 그 누구도 배포 이후에 함수를 수정하거나 예상치 못한 결과를 발생시키지 못한다.

### **외부 의존성**

lesson2에서 우리는 크립토키티 컨트랙트의 주소를 우리 `DApp`에서 직접 써넣었다. 그런데 만약 크립토키티 컨트랙트에 버그가 있었고, 누군가 모든 고양이들을 파괴해버렸다면 어떻게 될까?

그럴 일은 잘 없겠지만, 만약 그런 일이 발생한다면 우리의 `DApp`은 완전히 쓸모가 없어질 것이다. 우리 `DApp`은 주소를 코드에 직접 써넣기 때문에 얻던 고양이들도 받아올 수 없을 것이다. 우리 좀비들은 고양이를 먹을 수 없을 것이고, 우리는 그걸 고치기 위해 우리의 컨트랙트를 수정할 수도 없을 것이다.

이런 이유로, 대개의 경우 내 `DApp`의 중요한 일부를 수정할 수 있도록 하는 함수를 만들어놓는 것이 합리적일 것이다.

예를 들면 우리 `DApp`에 크립토키티 컨트랙트 주소를 직접 써넣는 것 대신, 언젠가 크립토키티 컨트랙트에 문제가 생기면 해당 주소를 바꿀 수 있도록 해주는 `setKittyContractAddress`함수를 만들 수 있을 것이다.

## **2. 소유 가능한 컨트랙트**

그렇다고 `setKittyContractAddress`함수를 `external`로 선언해주면 누구든 이 함수를 호출할 수 있기 때문에 아무나 크립토키티 컨트랙트의 주소를 바꿀 수 있게 되어서 앱을 무용지물로 만들어 버릴 수가 있다.

이런 경우를 대처하기 위해서, 최근에 주로 쓰는 하나의 방법은 컨트랙트를 `소유 가능`하게 만드는 것이다. 컨트랙트를 대상으로 특별한 권리를 가지는 소유자가 있음을 의미하는 것이다.

### **OpenZeppelin의 `Ownable` 컨트랙트**

아래에 나와있는 것은 `OpenZeppelin` 솔리디티 라이브러리에서 가져온 `Ownable` 컨트랙트이다. `OpenZeppelin` 은 `DApp` 에서 사용할 수 있는, 안전하고 커뮤니티에서 검증받은 스마트 컨트랙트의 라이브러리다.

```sol
/*
@title Ownable
@dev The Ownable contract has an owner address, and provides basic authorization control functions, this simplifies the implementation of "user permissions".
*/
contract Ownable {
    address public owner;
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /*
    @dev The Ownable constructor sets the original `owner` of the contract to the sender account.
    */
    function Ownable() public {
        owner = msg.sender;
    }

    /*
    @dev Throws if called by any account other than the owner.
    */
    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    /*
    @dev Allows the current owner to transfer control of the contract to a newOwner.
    @param newOwner The address to transfer ownership to.
    */
    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0));
        OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }
}
```

여기에 우리가 아직 본 적 없는 몇몇 새로운 요소가 있다 :

- 생성자(Constructor) : `function Ownable()`은 `생성자`이다. 컨트랙트와 동일한 이름을 가진, 생략할 수 있는 특별한 함수인 것이다. 이 함수는 컨트랙트가 생성될 때 딱 한 번만 실행된다.

- 함수 제어자(Function Modifier) : `modifier onlyOwner()`. 제어자는 다른 함수들에 대한 접근을 제어하기 위해 사용되는 일종의 유사 함수이다. 보통 함수 실행 전의 요구사항 충족 여부를 확인하는 데에 사용된다. `onlyOwner`의 경우에는 접근을 제한해서 오직 컨트랙트의 소유자만 해당 함수를 실행할 수 있도록 하기 위해 사용될 수 있다. 다음 챕터에서 함수 제어자에 대해 더 살펴보고 `_;`이 무엇을 하는 것인지도 알아볼 것이다.

- `indexed` 키워드 : 아직 필요하지 않으니 나중에 알아볼 것이다.

즉, `Ownable` 컨트랙트는 기본적으로 다음과 같은 것들을 한다 :

1. 컨트랙트가 생성되면 컨트랙트의 생성자가 `owner`에 `msg.sender`(컨트랙트를 배포한 사람)를 대입한다.
2. 특정한 함수들에 대해서 오직 `소유자`만 접근할 수 있도록 제한 가능한 `onlyOwner` 제어자를 추가한다.
3. 새로운 `소유자`에게 해당 컨트랙트의 소유권을 옮길 수 있도록 한다.

`onlyOwner`는 컨트랙트에서 흔히 쓰는 것 중 하나라, 대부분의 솔리디티 DApp들은 `Ownable`컨트랙트를 복사/붙여넣기 하면서 시작한다. 그리고 첫 컨트랙트는 이 컨트랙트를 상속해서 만든다.

## **3. onlyOwner 함수 제어자**

이제 기본 컨트랙트인 `ZombieFactory`가 `Ownable`을 상속하고 있으니, `onlyOwner`함수 제어자를 `ZombieFeeding`에서도 사용할 수 있게 되었다.

이건 컨트랙트가 상속되는 구조 때문이다. 아래 내용을 기억하자 :

```
ZombieFeeding is ZombieFactory
ZombieFactory is Ownable
```

그렇기 때문에 `ZombieFeeding` 또한 `Ownable`이고, `Ownable` 컨트랙트의 함수/이벤트/제어자에 접근할 수 있다. 이건 향후에 `ZombieFeeding`을 상속하는 다른 컨트랙트들에도 마찬가지로 적용된다.

### **함수 제어자**

함수 제어자는 함수처럼 보이지만, `function`키워드 대신 `modifier`키워드를 사용한다. 그리고 함수를 호출하듯이 직접 호출할 수 없다. 대신에 함수 정의부 끝에 해당 함수의 작동 방식을 바꾸도록 제어자의 이름을 붙일 수 있다.

`onlyOwner`를 살펴보면서 더 자세히 알아보자 :

```sol
/*
@dev Throws if called by any account other than the owner.
*/
modifier onlyOwner() {
    require(msg.sender == owner);
    _;
}
```

우리는 이 제어자를 다음과 같이 사용할 것이다 :

```sol
contract MyContract is Ownable {
    event LaughManiacally(string laughter);

    // 아래 `onlyOwner`의 사용 방법을 잘 보자 :
    function likeABoss() external onlyOwner {
        LaughManiacally("Muahahahaha");
    }
}
```

`likeABoss`함수의 `onlyOwner`제어자 부분을 보자. `likeABoss`함수를 호출하게 되면, `onlyOwner`의 코드가 먼저 실행된다. 그리고 `onlyOwner`의 `_;`부분을 `likeABoss`함수로 되돌아가 해당 코드를 실행하게 된다.

제어자를 사용할 수 있는 다양한 방법이 있지만, 가장 일반적으로 쓰는 예시 중 하나는 함수 실행 전에 `require`체크를 넣는 것이다.

`onlyOwner`의 경우에는, 함수에 이 제어자를 추가하면 오직 컨크랙트의 소유자만 이 해당 함수를 호출할 수 있다.

> *참고 : 이렇게 소유자가 컨트랙트에 특별한 권한을 갖도록 하는 것은 자주 필요하지만, 이게 악용될 수도 있다. 예를 들어, 소유자가 다른 사람의 좀비를 뺏어올 수 있도록 하는 백도어 함수를 추가할 수도 있는 것이다.*

> *그러니 이더리움에서 돌아가는 `DApp`이라고 해서 그것만으로 분산화되어 있다고 할 수는 없다. 반드시 전체 소스 코드를 읽어보고, 본인이 잠재적으로 걱정할 만한, 소유자에 의한 특별한 제어가 불가능한 상태인지 확인해야 한다. 개발자로서는 본인이 잠재적인 버그를 수정하고 `DApp`을 안정적으로 유지하도록 하는 것과, 사용자들이 그들의 데이터를 믿고 저장할 수 있는 소유자가 없는 플랫품을 만드는 것 사이에서 균형을 잘 잡는 것이 중요하다.*

## **4. 가스(Gas)**

### **가스 - 이더리움 DApp이 사용하는 연료**

솔리디티에서는 사용자들이 누군가가 만든 `DApp`의 함수를 실행할 때마다 `가스`라고 불리는 화폐를 지불해야한다. 사용자는 ETH를 이용해서 가스를 사기 때문에 누군가의 `DApp`함수를 실행하려면 사용자는 ETH를 소모해야만 한다.

함수를 실행하는 데에 얼마나 많은 가스가 필요한지는 그 함수의 로직(논리 구조)이 얼마나 복잡한지에 따라 달라진다. 각각의 연산은 소모되는 `가스 비용(gas cost)`이 있고, 그 연산을 수행하는 데에 소모되는 컴퓨팅 자원의 양이 이 비용을 결정한다. 예를 들어, `storage`에 값을 쓰는 것은 두 개의 정수를 더하는 것보다 훨씬 비용이 높다. 함수 전체 `가스 비용`은 그 함수를 구성하는 개별 연산들의 가스 비용을 모두 합친 것과 같다.

함수를 실행하는 것은 사용자들에게 실제 돈을 쓰게 하는 것이기 때문에, 이더리움에서 코드 최적화는 다른 프로그래밍 언어들에 비해 훨씬 더 중요하다. 만약 코드가 엉망이라면, 사용자들은 함수를 실행하기 위해 일종의 할증료를 더 내야 할 것이다. 그리고 수천 명의 사용자가 이런 불필요한 비용을 낸다면 할증료가 수십 억원까지 쌓일 수 있다.

### **가스가 왜 필요한가?**

이더리움은 크고 느린, 하지만 굉장히 안전한 컴퓨터와 같다고 할 수 있다.