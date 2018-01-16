
# 보안

스마트 컨트랙트는 한 번 코드가 올라가면 수정할 수 없고 소유자의 재산과 직결되기 때문에 보안 이슈에 굉장히 민감합니다. 스마트 컨트랙트 코드를 어떻게 짜느냐에 따라 생길 수 있는 여러 가지 보안 상의 취약점들이 이미 많이 알려져 있는 상황인데, 반드시 이런 보안 이슈에 대해 충분히 숙지하고 철저히 테스트를 한 뒤 코드를 메인 블록체인 네트워크에 배포하도록 합시다.

### 참고 자료

- [Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [A Survey of attacks on Ethereum smart contracts](https://eprint.iacr.org/2016/1007.pdf)
- [Security Considerations](https://solidity.readthedocs.io/en/develop/security-considerations.html#sending-and-receiving-ether)

## 기본 지침

스마트 컨트랙트 프로그래밍은 일반적인 타 영역에서의 프로그래밍과는 조금 다른 자세가 필요합니다. 어떻게 보면 하드웨어 프로그래밍에 가깝다고 볼 수 있는데, 보안 상의 취약점 혹은 버그가 가지는 비용이 굉장히 높고, 그걸 수정하기도 까다롭기 때문입니다. 따라서 단순히 알려져 있는 보안 취약점에 대해서만 대처하는게 아니라, 보안 상의 이슈가 생겨나지 않도록 근본적인 부분에서 신경을 써야할 필요가 있습니다. 기본적으로 아래와 같은 사항을 항상 신경씁시다.

- Prepare of failure : 아무리 신경을 쓰더라도 사람이 하는 일인 만큼 문제가 100% 생기지 않을 거라고 확신할 수 없습니다. 따라서, 뭔가 문제가 생기거나 버그가 발생했을 때 이에 대응하기 위한 방법이 필요합니다.
  * 문제가 생기면 컨트랙트를 중지시킬 수 있는 수단을 마련합시다(circuit breaker)
  * 버그 수정, 개선을 위한 효율적인 업그레이드 수단을 준비해둡시다.

- Roll out carefully : 최종 배포 전에 버그를 잡아내는게 가장 좋습니다.
  * 컨트랙트를 철저하게 테스트합니다. 새로운 공격 방법이 발견될 때마다 그에 해당하는 테스트 케이스를 반드시 추가합니다.
  * 사용 범위 및 테스트 범위를 나눠서 차근차근 출시를 진행합니다. 

- Keep contracts simple : 컨트랙트 코드가 복잡해질 수록 버그가 생길 확률도 올라갑니다.
  * 컨트랙트의 로직을 최대한 심플하게 유지합니다.
  * 컨트랙트와 함수를 최대한 작게 모듈화합니다.
  * 가능한 한 기존에 작성된 툴과 코드를 사용합니다.
  * 가능한 한 성능보다는 깔끔함을 중요시합니다.
  * 시스템에서 정확히 필요한 부분에만 블록체인을 사용합니다.

- Be aware of blockchain properties : 이더리움 프로그래밍에서 신경써야만 하는 몇가지 특징들이 있습니다.
  * 외부 컨트랙트 함수는 굉장히 신중하게 호출해야 합니다. 굉장히 악의적인 함수가 실행될 수 있고, 제어 로직을 변경시킬 수 있기 때문입니다.
  * 퍼블릭 함수들은 악의적으로 호출될 수 있고, private data역시 누구나 열람할 수 있습니다.
  * gas cost와 gas limit를 항상 신경씁시다.


## 알려진 공격

### Race Conditions

#### Reentrancy

처음 함수 호출이 이뤄진 후 해당 함수의 실행이 끝나기 전에 동일한 함수를 반복적으로 실행했을 때 문제가 생기는 경우입니다.

```javascript
//INSECURE
mapping (address => uint) private userBalances;

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    require(msg.sender.call.value(amountToWithdraw)());
    userBalances[msg.sender] = 0;
}
```

함수가 종료되기 전에 사용자의 잔고가 0으로 설정되지 않기 때문에, 두번째 함수 호출까지 성공하게 됩니다. 따라서 함수를 계속해서 반복적으로 호출할 경우 사용자의 잔고보다 훨씬 더 많은 양을 인출해갈 수 있게 됩니다. 이러한 공격을 막기 위해 외부 함수를 호출하기 전에 내부 상태를 먼저 수정해주어야 합니다.

```javascript
mapping (address => uint) private userBalances;

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    userBalances[msg.sender] = 0;
    require(msg.sender.call.value(amountToWithdraw)());
}
```

#### Cross-function Race Conditions

위와 유사하게, 같은 상태를 공유하는 서로 다른 두 함수를 이용한 공격이 가능합니다.

```javascript
//INSECURE
mapping (address => uint) private userBalances;

function transfer(address to, uint amount) {
    if(userBalances[msg.sender] >= amount) {
        userBalances[to] += amount;
        userBalances[msg.sender] -= amount;
    }
}

function withdrawBalance() public {
    uint amountToWithdraw = userBalances[msg.sender];
    require(msg.sender.call.value(amountToWithdraw)());
    userBalances[msg.sender] = 0;
}
```
이 경우 마찬가지로 msg.sender.call.value(amountToWithdraw)()가 실행되는 중에 transfer를 실행해버리면 이미 인출해간 돈을 다른 사람의 계좌에 보내는 게 가능해집니다.

#### Pitfalls in Race Condition Solutions

일반적으로 race condition 공격에 대응하기 위해 1. 먼저 내부적인 처리를 끝마치고, 2. 외부 함수를 호출한다. 순으로 코드를 작성하는 걸 추천합니다. 이런 룰을 잘 지키면 일반적으로 race condition을 회피할 수 있는데, 주의해야할 점은 단순히 외부 함수 호출 외에 외부함수를 호출하는 함수 역시도 일찍 호추하는 걸 피해야 합니다. 아래와 같은 예시 상황이 있습니다.

```javascript
//INSECURE
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function withdraw(address recipient) public {
    uint amountToWithdraw = userBalances[recipient];
    rewardsForA[recipient] = 0;
    require(recipient.call.value(amountToWithdraw)());
}

function getFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]);

    rewardsForA[recipient] += 100;
    withdraw(recipient);
    claimedBonus[recipient] = true;
}
```

여기서 getFirstWithdrawBonus가 호출하는 withdraw 함수는 그 자체로는 일반 함수지만, 내부에서 다시 외부 함수를 호출합니다. 따라서, 이러한 함수를 호출하는 것 역시 가장 마지막에 진행되어야하고 이런 함수는 untrusted한 함수로 생각해야 합니다.

```javascript
mapping (address => uint) private userBalances;
mapping (address => bool) private claimedBonus;
mapping (address => uint) private rewardsForA;

function untrustedWithdraw(address recipient) public {
    uint amountToWithdraw = userBalances[recipient];
    rewardsForA[recipient] = 0;
    require(recipient.call.value(amountToWithdraw)());
}

function untrustedGetFirstWithdrawalBonus(address recipient) public {
    require(!claimedBonus[recipient]);

    claimedBonus[recipient] = true;
    rewardsForA[recipient] += 100;
    untrustedWithdraw(recipient);
}
```

### Transaction-Ordering Dependence(TOD) / Front Running

위의 예시는 공격자가 한 트랜잭션 내에서 악의적인 코드를 실행하는 경우의 예시입니다. 이 외에도 race condition에 관한 다른 종류의 공격이 있는데, 그건 바로 '블록 내에서 트랜잭션의 순서는 쉽게 조작가능하다'라는 사실을 이용하는 것입니다. 트랜잭션이 실제 블록에 포함되기 전에 어떤 액션이 일어날 지 알 수 있기 때문에, 일부 종류의 컨트랙트에서는 문제가 발생할 수 있습니다. 이런 종류의 공격을 방지하는 건 굉장히 어렵습니다.

### Timestamp Dependence

블록 상의 타임스탬프 값은 마이너가 조작할 수 있는 값이기 때문에, 이 값은 신중하게 사용해야 합니다.

```javascript
uint someVariable = now + 1;

if (now % 2 == 0) { // the now can be manipulated by the miner

}

if ((someVariable - 100) % 2 == 0) { // someVariable can be manipulated by the miner

}
```

### Integer Overflow and Underflow
정수 연산 과정에서 오버플로우 혹은 언더플로우가 일어날 수 있는지 잘 체크해봐야 합니다. 

### DoS with (Unexpected) revert

```javascript
// INSECURE
contract Auction {
    address currentLeader;
    uint highestBid;

    function bid() payable {
        require(msg.value > highestBid);

        require(currentLeader.send(highestBid)); // Refund the old leader, if it fails then revert

        currentLeader = msg.sender;
        highestBid = msg.value;
    }
}
```

bid 값이 현재 highest보다 높아지면 기존 것을 환불해주고 새로 갱신해주는데, 이 경우 악의적인 공격자가 한 번 highest가 된 후 모든 refund가 반드시 실패하게 만들면 영원히 leader가 될 수 있습니다. 

```javascript
address[] private refundAddresses;
mapping (address => uint) public refunds;

// bad
function refundAll() public {
    for(uint x; x < refundAddresses.length; x++) { // arbitrary length iteration based on how many addresses participated
        require(refundAddresses[x].send(refunds[refundAddresses[x]])) // doubly bad, now a single failure on send will hold up all funds
    }
}
```

위와 같은 예시도, 전체 중에 한 명만 refund가 실패하더라도 누구도 refund를 받을 수 없게 됩니다.


### DoS with Block Gas Limit

위의 예시에서 추가적으로, gas limit를 이용한 공격 역시 가능합니다. 아주 소량의 refund가 필요한 굉장히 여러 개의 주소를 등록해버리면, 모두 refund할 수 없을 만큼 gas 필요량이 높아져서 아무도 refund를 받을 수 없게 될 것입니다.

만약에 전체 길이를 알 수 없는 배열에 대한 반복문이 반드시 필요하다면, 그 전체 반복을 한 트랜잭션에서 모두 수행하는 게 아니라 여러 개의 트랜잭션으로 나눠서 수행하는게 좋습니다.

```javascript
struct Payee {
    address addr;
    uint256 value;
}

Payee[] payees;
uint256 nextPayeeIndex;

function payOut() {
    uint256 i = nextPayeeIndex;
    while (i < payees.length && msg.gas > 200000) {
      payees[i].addr.send(payees[i].value);
      i++;
    }
    nextPayeeIndex = i;
}
```

### Forcibly Sending Ether to a Contract

fallback 함수를 호출하지 않고도 특정 컨트랙트에 강제로 이더리움을 보낼 수 있습니다. 만약에 컨트랙트의 fallback 함수에 중요한 로직을 둬야 하거나 현재 컨트랙트의 잔고에 기반한 계산을 만들어야할 경우 이러한 상황을 염두에 둬야 합니다.

```javascript
contract Vulnerable {
    function () payable {
        revert();
    }

    function somethingBad() {
        require(this.balance > 0);
        // Do something bad
    }
}
```

fallback이 반드시 실패하기 때문에 somethingBad가 실행될 일이 없다고 생각할 수 있지만 위에서 말한 것처럼 이런 상황이 충분히 발생하게 될 수 있습니다. 일반적으로, 내 컨트랙트에 이더리움이 전송되는 것에 제약을 걸 수 없다고 생각해야 합니다.