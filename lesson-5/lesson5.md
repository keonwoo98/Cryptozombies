# **Cryptozombies[lesson5]**

> **크립토좀비로 솔리디티 공부하기** [링크](https://cryptozombies.io)

## **1. 이더리움 상의 토큰**

이더리움에서 `토큰`(ERC20 토큰)은 기본적으로 그저 몇몇 공통 규약을 따르는 스마트 컨트랙트이다. 즉, 다른 모든 토큰 컨트랙트가 사용하는 표준 함수 집합을 구현하는 것이다. 예를 들면, `transfer(address _to, uint256 _value)`나 `balanceOf(address _owner)`같은 함수들이 있다.

내부적으로 스마트 컨트랙트는 보통 `mapping(address => uint256) balances`와 같은 매핑을 가지고 있다. 각각의 주소에 잔액이 얼마나 있는지 기록하는 것이다.

즉, 기본적으로 토큰은 그저 하나의 컨트랙트이다. 그 안에서 누가 얼마나 많은 토큰을 가지고 있는지 기록하고, 몇몇 함수를 가지고 사용자들이 그들의 토큰을 다른 주소로 전송할 수 있게 해주는 것이다.

### **왜 이렇게 해야 하나요?**

모든 `ERC20 토큰`들이  똑같은 이름의 동일한 함수 집합을 공유하기 때문에, 이 토큰들에 똑같은 방식으로 상호작용이 가능하다.

즉, 하나의 `ERC20 토큰`과 상호작용할 수 있는 애플리케이션 하나를 만들면, 이 앱이 다른 어떤 `ERC20 토큰`과도 상호작용이 가능한 것이다. 이런 방식으로 앱에 더 많은 토큰들을 추가할 수 있다. 커스텀 코드를 추가하지 않고서도 말이다. 그저 새로운 토큰의 컨트랙트 주소만 끼워넣으면 된다. 그리고 나면 앱에서 사용할 수 있는 또 다른 토큰이 생기는 것이다.

이러한 것의 한 예로는 거래소가 있다. 한 거래소에서 새로운 `ERC20 토큰`을 상장할 때, 실제로는 이 거래소에서 통신이 가능한 또 하나의 스마트 컨트랙트를 추가하는 것이다. 사용자들은 이 컨트랙트에 거래소의 지갑 주소에 토큰을 보내라고 할 수 있고, 거래소에서는 이 컨트랙트에 사용자들이 출금을 신청하면 토큰을 다시 돌려보내라고 할 수 있게 만드는 것이다.

거래소에서는 이 전송 로직을 한 번만 구현하면 된다. 그리고서 새로운 `ERC20 토큰`을 추가하고 싶으면, 데이터베이스에 단순히 새 컨트랙트 주소를 추가하기만 하면 되는 것이다.

### **다른 토큰 표준**

`ERC20 토큰`은 화폐처럼 사용되는 토큰으로는 적절하다. 하지만 우리의 좀비 게임에서 좀비를 표현할 때에는 그다지 쓸모 있진 않다.

첫째로, 좀비는 화폐처럼 분할할 수가 없다. 내가 누군가에게 `0.237ETH`를 보낼 수는 있지만, 0.237개의 좀비를 보낼 수는 없다.

둘째로, 모든 좀비가 똑같지는 않다. 나의 좀비와 다른 어느 사용자의 좀비는 완전히 다르다.

여기에 크립토좀비와 같은 크립토 수집품을 위해 더 적절한 토큰 표준이 있다. 바로 `ERC721 토큰`이다.

`ERC721 토큰`은 대체 불가능하다. 각각의 토큰이 유일하고 분할이 불가하기 때문이다. 이 토큰은 하나의 전체 단위로만 거래할 수 있고, 각각의 토큰은 고유한 `ID`를 가지고 있다. 때문에 이 방법이 좀비를 거래할 수 있게 하기에는 가장 적절하다.

> `ERC721`과 같은 표준을 사용하면 우리의 컨트랙트에서 사용자들이 우리의 좀비를 거래/판매할 수 있도록 하는 경매나 중계 로직을 우리가 직접 구현하지 않아도 된다는 이점이 있다. 우리가 스펙에 맞추기만 하면, 누군가 `ERC721` 자산을 거래할 수 있도록 하는 거래소 플랫폼을 만들면 우리의 `ERC721` 좀비들을 그 플랫폼에서 쓸 수 있게 될 것이다. 자신만의 거리 로직을 만드느라 고생하는 것보다 토큰 표준을 사용하는 것이 명확한 이점이 있다는 것이다.

## **2. ERC721 표준, 다중 상속**

`ERC721` 표준을 한번 같이 살펴보도록 하자 :
```sol
contract ERC721 {
	event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
	event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);
	
	function balanceOf(address _owner) public view returns (uint256 _balance);
	function ownerOf(uint256 _tokenId) public view returns (address _owner);
	function transfer(address _to, uint256 _tokenId) public;
	function approve(address _to, uint256 _tokenId) public;
	function takeOwnership(uint256 _tokenId) public;
}
```

위 내용이 앞으로 남은 챕터들에서 구현해 나갈 메소드들의 목록이다.

> 참고 : `ERC721` 표준은 현재 초안인 상태이고, 아직 공식적으로 채택된 구현 버전은 없다. 이 튜토리얼에서는 `OpenZeppelin` 라이브러리에서 쓰이는 현재 버전을 사용할 것이지만, 공식 릴리즈 이전에 언젠가 바뀔 가능성도 있다. 그러니 하나의 구현 가능한 버전 정도로만 생각하고, `ERC721 토큰`의 정식 표준으로 생각하지는 말자.

### **토큰 컨트랙트 구현하기**

토큰 컨트랙트를 구현할 때, 처음 해야 할 일은 바로 인터페이스를 솔리디티 파일로 따로 복사하여 저장하고 `import "./erc721.sol;`을 써서 import를 하는 것이다. 그리고 해당 컨트랙트를 상속하는 우리의 컨트랙트를 만들고, 각각의 함수를 오버라이딩하여 정의하여야 한다.

그런데 여기서 잠깐 - `ZombieOwnership`은 이미 `ZombieAttack`을 상속하고 있다. 그렇다면 어떻게 `ERC721`도 상속하게 할 수 있을까?

솔리디티에서는 다음과 같이 한 컨트랙트가 다수의 컨트랙트를 상속할 수 있다 :
```sol
contract SatoshiNakamoto is NickSzabo, HalFinney {

}
```

다중 상속을 쓸 때는 상속하고자 하는 다수의 컨트랙트를 쉼표(`,`)로 구분하면 된다.

## **3. balanceOf & ownerOf**

### **balanceOf**

```sol
function balanceOf(address _onwer) public view returns (uint256 _balance);
```

이 함수는 단순히 `address`를 받아, 해당 `address`가 토큰을 얼마나 가지고 있는지 반환한다. 이 경우, 우리의 `토큰`은 좀비다.

### **ownerOf**

```sol
function ownerOf(uint256 _tockenId) public view returns (address _owner);
```

이 함수에서는 `토큰 ID`(우리의 경우, 좀비 ID)를 받아, 이를 소유하고 있는 사람의 `address`를 반환한다.

이 함수들은 구현하기가 매우 수월하다. 이 정보를 저장하는 `mapping`을 우리 `DApp`에 이미 가지고 있기 때문이다. 위에서 봤듯이, 이 함수들은 단 한 줄로 구현할 수 있다.

> 참고 : `uint256`은 `uint`와 동일하다는 것을 기억하자. 우리 코드에서 지금까지 `uint`를 사용해왔지만, 여기서는 우리가 스펙을 복사/붙여넣기 했으니 `uint256`을 쓸 것이다.

## **4. 리팩토링**

이전 챕터에서 `ownerOf`라는 함수를 정의했다. 하지만 `lesson4`에서 우리는 `zombiefeeding.sol`에서 `ownerOf`와 똑같은 이름의 `modifier`를 만들었었다. 때문에 컴파일 시 에러가 날 것이다.

그렇다면 `ZombieOwnership`의 함수 이름을 이름을 다른 걸로 바꿔야 할까?

아니다. 우리는 `ERC721 토큰` 표준을 사용하고 있다. 이 말은 즉 다른 컨트랙트들이 우리의 컨트랙트가 정확한 이름으로 정의된 함수들을 가지고 있을 것이라 예상한다는 것이다. 그게 바로 이런 표준이 유용하게끔 하는 것이니 말이다. 만약 우리 컨트랙트가 `ERC721`을 따른다는 것을 다른 컨트랙트가 안다면, 이 다른 컨트랙트는 우리의 내부 구현 로직을 모르더라도 우리와 통신할 수 있다.

그러니 `ownerOf` 함수의 이름을 바꾸는 것이 아니라, `lesson4`에서 만든 `modifer`의 이름을 다른 것으로 바꾸도록 리팩토링을 해야 한다.

## **5. ERC721 : 전송 로직**

이제 우리의 `ERC721`에서 한 사람이 다른 사람에게 소유권을 넘기는 것을 구현해 나갈 것이다.

`ERC721` 스펙에서는 토큰을 전송할 때 2개의 다른 방식이 있음을 기억하자 :
```sol
function transfer(address _to, uint256 _tockenId) public;
function approve(address _to, uint256 _tockenId) public;
function takeOwnership(uint256 _tockenId) public;
```

1. 첫 번째 방법은 토큰의 소유자가 전송 상대의 `address`, 전송하고자 하는 `_tockenId`와 함께 `transfer`함수를 호출하는 것이다.

2. 두 번째 방법은 토큰의 소유자가 먼저 위에서 본 정보들을 가지고 `approve`를 호출하는 것이다. 그리고서 컨트랙트에 누가 해당 토큰을 가질 수 있도록 허가를 받았는지 저장한다. 보통 `mapping (uint256 => address)`를 써서 말이지. 이후 누군가 `takeOwnership`을 호출하면, 해당 컨트랙트는 이 `msg.sender`가 소유자로부터 토큰을 받을 수 있게 허가를 받았는지 확인한다. 그리고 허가를 받았다면 해당 토큰을 그에게 전송한다.

사실 `transfer`와 `takeOwnership` 모두 동일한 전송 로직을 가지고 있다. 순서만 반대인 것이다. 전자는 토큰을 보내는 사람이 함수를 호출하는 것이고 후자는 토큰을 받는 사람이 호출하는 것이다.

그러니 이 로직만의 프라이빗 함수, `_transfer`를 만들어 추상화하는 것이 좋을 것이다. 두 함수에서 모두 쓸 수 있도록 말이다. 이렇게 하면 똑같은 코드를 두 번씩 쓰지 않아도 된다.

### **`ERC721` : Transfer**

```sol
function _transfer(address _from, address _to, uint256 _tokenId) private {
	ownerZombieCount[_to]++;
	ownerZombieCount[_from]--;
	zombieToOwner[_tokenId] = _to;
	Transfer(_from, _to, _tokenId);
}

function transfer(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
	_transfer(msg.sender, _to, _tokenId);
}
```

### **`ERC721` : Approve**

`approve` / `takeOwnership`을 사용하는 전송은 2단계로 나뉜다 :
1. 원래 소유자가 새로운 소유자의 `address`와 그에게 보내고 싶은 `_tokenId`를 사용하여 `approve`를 호출한다.

2. 새로운 소유자가 `_tokenId`를 사용하여 `takeOwnership`함수를 호출하면, 컨트랙트는 그가 승인된 자인지 확인하고 그에게 토큰을 전송한다.

이처럼 2번의 함수 호출이 발생하기 때문에, 우리는 함수 호출 사이에 누가 무엇에 대해 승인이 되었는지 저장할 데이터 구조가 필요할 것이다.

```sol
function approve(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
	zombieApprovals[_tokenId] = _to;
	Approval(msg.sender, _to, _tokenId);
}
```

### **`ERC721` : takeOwnership**

`takeOwnership`함수에서는 `msg.sender`가 이 토큰/좀비를 가질 수 있도록 승인되었는지 확인하고, 승인이 되었다면 `_transfer`를 호출해야 한다.

```sol
function takeOwnership(uint256 _tokenId) public {
	require(zombieApprovals[_tokenId] == msg.sender);
	address owner = ownerOf(_tokenId);
	_transfer(owner, msg.sender, _tokenId);
}
```

## **6. 오버플로우 막기**

### **컨트랙트 보안 강화 : 오버플로우와 언더플로우**

스마트 컨트랙트를 작성할 때 인지하고 있어야 할 하나의 주요한 보안 기능은 오버플로우와 언더플로우를 막는 것이다.

`uint8`에 저장할 수 있는 가장 큰 수는 이진수로 `11111111` (10진수로 2^8 - 1 = 255)이다. 만약 `11111111`에 1을 더하면 이 수는 `00000000`으로 돌아간다. 이것이 오버플로우이다.

반대로 `0`에서 1을 빼면 255와 같아지는 것은 언더플로우이다. (`uint`에 부호가 없기 때문에 음수는 될 수 없다.)

`uint256`을 쓴다면 오버플로우가 발생하지 않을 것 같지만 `DApp`에 예상치 못한 문제가 발생하지 않도록 컨트랙트에 보호 장치를 두는 것이 좋다.

### **SafeMath 사용하기**

이를 막기 위해, `OpenZeppelin`에서 기본적으로 이런 문제를 막아주는 `SafeMath`라고 하는 `라이브러리`를 만들었다.

`라이브러리(Library)`는 솔리디티에서 특별한 종류의 컨트랙트이다. 이게 유용하게 사용되는 경우 중 하나는 기본(native) 데이터 타입에 함수를 붙일 때이다.

예를 들어, `SafeMath` 라이브러리를 쓸 때는 `using SafeMath for uint256`이라는 구문을 사용할 것이다. `SafeMath` 라이브러리는 4개의 함수를 가지고 있다. `add`, `sub`, `mal`, `div` 이다. 그리고 이제 우리는 `uint256`에서 다음과 같이 이 함수들에 접근할 수 있다.

```sol
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```

## 7. SafeMath

`SafeMath` 내부 코드 :

```sol
library SafeMath {
	
	function mul(uint256 a, uint256 b) internal pure returns (uint256) {
		if (a == 0) {
			return 0;
		}
		uint256 c = a * b;
		assert(c / a == b);
		return c;
	}

	function div(uint256 a, uint256 b) internal pure returns (uint256) {
		// assert(b > 0);
		// Solidity automatically throws when dividing by 0
		uint256 c = a / b;
		// assert(a == b * c + a % b);
		// There is no case in which this doesn't hold
		return c;
	}

	function sub(uint256 a, uint256 b) internal pure returns (uint256) {
		assert(b <= a);
		return a - b;
	}

	fundtion add(uint256 a, uint256 b) internal pure returns (uint256) {
		uint256 c = a + b;
		assert(c >= a);
		return c;
	}
}
```

먼저 `library` 키워드를 보자. 라이브러리는 `contract`와 비슷하지만 조금 다른 점이 있다. 우리의 경우에 라이브러리는 우리가 `using`키워드를 사용할 수 있게 해준다. 이를 통해 라이브러리의 메소드들을 다른 데이터 타입에 적용할 수 있다.

```sol
using SafeMath for uint;
// 우리는 이제 이 메소드를 아무 uint에서나 쓸 수 있다.
uint test = 2;
test = test.mul(3); // test == 6
test = test.add(5); // test == 11
```

`mul`과 `add` 함수는 각각 2개의 인수를 필요로 한다. 하지만 우리가 `using SafeMath for uint`를 선언할 때, 우리가 함수를 적용하는 `uint(test)`는 첫 번째 인수로 자동으로 전달된다.

`SafeMath`가 어떤 것을 하는지 보기 위해 `add`함수의 내용을 살펴보자 :

```sol
function add(uint256 a, uint256 b) internal pure returns (uint256) {
	uint256 c = a + b;
	assert(c >= a);
	return c;
}
```

기본적으로 `add`는 그저 2개의 `uint`를 `+`처럼 더한다. 하지만 그 안에 `assert` 구문을 써서 그 합이 `a`보다 크도록 보장한다. 이것이 오버플로우를 막아주는 것이다.

`assert`는 조건을 만족하지 않으면 에러를 발생시킨다는 점에서 `require`와 비슷하다. `assert`와 `require`의 차이점은, `require`는 함수 실행이 실패하면 남은 가스를 사용자에게 되돌려 주지만, `assert`는 그렇지 않다는 것이다. 즉 대부분은 `require`를 쓰고 싶을 것이다. `assert`는 일반적으로 코드가 심각하게 잘못 실행될 때 사용한다. (`uint` 오버플로우)

간단히 말해, `SafeMath`의 `add`, `sub`, `mul`, `div`는 4개의 기본 수학 연산을 하는 함수이지만, 오버플로우나 언더플로우를 발생하면 에러를 발생시키는 것이다.

### **현재 코드에 `SafeMath` 사용하기**

오버플로우나 언더플로우를 막기 위해, 코드에서 `+`, `-`, `*`, `/`을 쓰는 곳을 찾아 `add`, `sub`, `mul`, `div`로 교체한다.

예를 들어 :
```sol
myUint++;
```
대신
```sol
myUint = myUint.add(1);
```

작은 문제가 하나있다. `uint256`으로 선언된 데이터 타입은 위와 같이 수정해주면 되지만 `uint32`나 `uint16`으로 선언된 데이터 타입은 `SafeMath`의 `add` 메소드를 사용하면, 이 타입들을 `uint256`으로 바꿀 것이기 때문에 실제로 오버플로우를 막을 수 없다.

이는 즉 `uint16` 과 `uint32`에서 오버플로우 / 언더플로우를 막기 위해 2개의 라이브러리를 더 만들어야 한다는 것을 의미한다. `SafeMath`와 정확히 같은 방식으로 `SafeMath16`과 `SafeMath32`를 데이터 타입만 바꿔서 만들었다.

## **8. 주석**

솔리디티에서 주석을 다는 것은 자바스크립트와 비슷하다.

한 줄 주석 :  
`// 주석 내용`  

여러 줄 주석 :  
`/*`  
`주석`    
`내용 `  
`*/`  

솔리디티 커뮤니티에서 표준으로 쓰이는 형식은 `natspec`이라 불린다. 아래와 같이 생겼다 :
```SOL
/// @title 기본적인 산수를 위한 컨트랙트
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice 지금은 곱하기 함수만 추가한다.
contract Math {
	/// @notice 2개의 숫자를 곱한다.
	/// @param x 첫 번쨰 uint.
	/// @param y 두 번째 uint.
	/// @return z (x * y) 곱의 값
	/// @dev 이 함수는 현재 오버플로우를 확인하지 않는다.
	function multiply(uint x, uint y) returns (uint z) {
		// 이것은 일반적인 주석으로, natspec에 포함되지 않는다.
    	z = x * y;
	}
}
```

`@notice`는 사용자에게 컨트랙트/함수가 무엇을 하는지 설명한다.

`@dev`는 개발자에게 추가적인 상세 정보를 설명하기 위해 사용한다.

`@param`과 `@return`은 함수에서 어떤 매개 변수와 반환값을 가지는지 설명한다.

