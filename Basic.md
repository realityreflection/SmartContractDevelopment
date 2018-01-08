
# 기본 코딩

Solidity의 기본적인 문법 체계에 대해 배워봅시다. 자세한 내용은 [Soldity document](https://solidity.readthedocs.io/en/latest/index.html)에서 확인하실 수 있습니다.

Solidity는 .sol 파일 확장자를 사용합니다.

## version pragma

solidity 파일의 맨 첫번째 줄에는 version pragma 내용이 들어가게 됩니다. 이는 현재 작성된 solidity 코드가 어떤 버전의 컴파일러를 기준으로 작성되었는지 나타내기 위한 것입니다. 현재 최신 버전 기준으로,

```javascript
pragma solidity ^0.4.18
```

다른 파일에 작성된 코드를 불러오기 위해서는 import 를 사용합니다. 

```javascript
import 'otherFile.sol`
```

주석은 C/C++과 유사하게 한 줄 주석에 `//`, 여러 줄 주석에 `/**/`를 사용합니다.

## Contract

컨트랙트는 객체 지향 언어에서의 클래스와 유사하다고 생각하시면 됩니다. 각각의 컨트랙트는 상태 변수(State Variables), 함수(Function), 함수 모디파이어(Function Modifier), 이벤트(Event), 구조체(Struct Types), 열거 타입(Enum Types) 등을 포함할 수 있고 상속도 가능합니다. 여기서 작성된 컨트랙트가 실제 이더리움 실행 환경 상에서의 하나의 컨트랙트를 의미하게 된다고 생각하시면 됩니다.

contract를 작성하는 기본 문법은 다음과 같습니다.

```javascript
pragma solidity ^0.4.18;

contract contractName {
    // contract 내용
}
```

### types

기본적인 값 타입에는 다음과 같은 타입들이 있습니다.

- bool : true / false 값을 가지는 진리값 타입
- int / uint : 사이즈에 따라 8부터 8단위로 256까지의 숫자가 붙습니다. int8, uint8, int16, uint16, ... int256, uint256
- address : 이더리움 주소 값 타입입니다.

```javascript
pragma solidity ^0.4.18;

contract contractName {
    // 이 컨트랙트가 가지게 되는 두 가지 상태값
    uint256 state1;
    int256 state2;
}
```

### Function

함수는 다음과 같은 형태로 선언하게 됩니다.

```javascript
function functionName(type1 name1, type2 name2, ...) {internal|external} [pure|constant|view|payable] returns (returnType) {
    //function 내용
}
```

함수 뒤쪽에 붙는 각종 옵션 및 modifier들이 많아서 함수 선언이 굉장히 복잡합니다. 복잡하게 생각할 필요는 없고 필요할 때 찾아보고 의미에 맞는 걸 쓰면 됩니다. 이후 다양한 예시를 통해 차근차근 공부해봅시다.

### mapping

mapping 타입은 아래와 같이 선언해서 사용합니다.

```javascript
mapping(_KeyType => _ValueType)
```

mapping은 해시 테이블과 유사하게 생각하면 됩니다.

```javascript
pragma solidity ^0.4.18;

contract MappingExample {
    mapping(address => uint256) public balances;

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }
}
```

### event

이벤트는 트랜잭션에 대한 로그라고 생각하시면 됩니다. 어떤 트랜잭션이 발생할 때 이벤트를 실행하면, 블록체인 외부의 클라이언트 사이드에서 그런 트랜잭션이 발생했다는 걸 쉽게 추적할 수 있게 됩니다. 이벤트 자체를 컨트랙트 내부에서 사용하지는 않습니다.

```javascript
contract EventTest {
    event EventTest(address indexed from);

    function testCall(address _from) public {
        EventTest(_from);
    }
}
```

위와 같이 이벤트를 만들어두면 클라이언트 사이드에서 web3(이더리움 환경과 통신할 수 있는 라이브러리)를 이용해 특정 이벤트가 발생했는지 쉽게 확인할 수 있습니다. 

### function modifier

function modifier는 함수의 동작을 쉽게 편경하기 위해 사용됩니다. 예를 들어, 특정 함수(트랜잭션)이 어떤 조건을 만족할 때에만 실행되게 수정하고 싶을 때 function modifier를 사용할 수 있습니다.

```javascript
contract Ownable {
    address owner;

    Ownable() public {
        owner = msg.sender;
    }

    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
}

contract Test is Ownable {
    event Call();

    function ownCall() public onlyOwner {
        Call();
    }
}
```

require 혹은 assert는 인자로 넘어온 값이 true가 아닐 경우 트랜잭션을 실패하게 만듭니다. 일반적으로 외부 인자가 조건을 만족하는지 체크할 때 require, 클래스 내부의 값이 정상적인 조건을 만족하는지 확인할 때 assert를 씁니다. ownCall에서 볼 수 있듯이 modifier를 함수 뒤쪽에 붙이면 해당 함수의 동작이 바뀝니다. 기본적으로 modifier 내부의 동작을 모두 실행한 다음, _ 부분에서 함수의 본체 내용을 실행하게 됩니다. 하나의 함수에 여러 개의 modifier를 사용할 수 있습니다.

그리고 위 코드에서 msg라는 변수를 볼 수 있는데, 이는 solidity 환경에서 제공되는 글로벌 변수입니다. msg 변수는 해당 트랜잭션을 실행시킬 때 넘어온 메시지 데이터 정보를 담고 있습니다. 보통 아래 두 가지가 많이 쓰입니다.

```
- msg.sender : 해당 메시지의 발신자
- msg.value : 같이 전송된 이더의 양(wei = 1/10^18 이더리움 기준. 이더리움 최소 거래 단위)
```

msg 외에도 많이 쓰이는 글로벌 변수에는 다음과 같은 것들이 있습니다.
```
- this : 현재 contrat
- super : 현재 contract의 부모
- now : 현재 시간(utc epoch 기준)
```

그 외에, 이더리움 단위 계산 등을 편하게 하기 위해 다음과 같은 방법들이 사용됩니다.

```javascript
uint256 e = 1 * 1 ether;  // 10^18
uint256 f = 1 * 1 finney; // 10^15
uint256 s = 1 * 1 szabo;  // 10^12
```

문법에 대해 궁금한 내용을 빨리 확인하고 싶을 때에는 [cheatsheet](https://solidity.readthedocs.io/en/latest/miscellaneous.html#cheatsheet)을 보면 편합니다.

### 보안

스마트 컨트랙트는 한 번 배포되면 수정할 수 없고 잘못 작성될 경우 자신의 이더 잔고가 모조리 날아가버리는 등의 사태가 발생할 수 있기 때문에 보안이 굉장히 중요합니다. 여러 종류의 이미 알려진 공격 방식들이 있고 그런 공격 방식에 대한 대응법 및 좋은 코딩 스타일이 존재합니다. 이 내용은 [Security Considerations](https://solidity.readthedocs.io/en/latest/security-considerations.html#)에서 확인할 수 있고 코드를 짜기 전 꼭 한 번 읽어보는게 좋습니다.