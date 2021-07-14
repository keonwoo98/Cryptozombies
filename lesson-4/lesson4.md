# **Cryptozombies[lesson4]**

> **크립토좀비로 솔리디티 공부하기** [링크](https://cryptozombies.io)

## **1. Payable**

지금까지 꽤 많은 `함수 제어자(function modifier)`르 다뤘다. 한번 빠르게 복습해보자.

1. 우린 함수가 언제, 어디서 호출될 수 있는지 제어하는 접근 제어자(visibility modifier)를 알게 되었다 :  
`private`은 컨트랙트 내부의 다른 함수들에서만 호출될 수 있음을 의미한다. `internal`은 `private`과 비슷하지만, 해당 컨트랙트를 상속하는 컨트랙트에서도 호출될 수 있다. `external`은 오직 컨트랙트 외부에서만 호출될 수 있다. 마지막으로 `public`은 내외부 모두에서, 어디서든 호출될 수 있다.