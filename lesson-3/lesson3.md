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

