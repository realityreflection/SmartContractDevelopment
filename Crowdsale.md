
# StandardToken

ERC-20 규약은 [여기](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md)서 확인할 수 있습니다. 이런 ERC-20 규약을 일일히 구현하는 게 굉장히 귀찮은 일인데, 다행히 [zeppelin-solidity](https://github.com/OpenZeppelin/zeppelin-solidity)라는 프로젝트가 있어 대부분의 기본 컨트랙트는 이 프로젝트에서 가져다 쓰고, 필요한 부분만 만들면 됩니다.

일단 ERC-20 규약이 기본적으로 갖춰야 하는 인터페이스는 다음과 같습니다.

```javascript
contract ERC20Interface {
    function totalSupply() public constant returns (uint);
    function balanceOf(address tokenOwner) public constant returns (uint balance);
    function allowance(address tokenOwner, address spender) public constant returns (uint remaining);
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);
 
    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```

이 인터페이스의 작성 예시는 [ERC20Basic](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/ERC20Basic.sol) 및 [ERC20](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/ERC20.sol), [BasicToken](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/BasicToken.sol), [StandardToken](https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/token/StandardToken.sol) 에서 확인해보시면 됩니다.

# Crowdsale

토큰을 만들고 나면 해당 토큰을 이용한 crowdsale 역시 쉽게 만들 수 있습니다. 이 부분도 위에 링크한 zeppelin-solidity에 기본 예시 코드들이 굉장히 잘 작성되어 있으므로 그 내용을 참고하면 됩니다.

crowdsale은 일반적으로 crowdsale을 진행하는 컨트랙트에 이더를 입금하면 그 이더에 대응되는 토큰을 해당 주소로 지급해주는 방식으로 만들어집니다. 여기서 몇 종류의 파생이 존재하는데, 대략적으로 아래와 같이 분류할 수 있습니다.

1. cap이 있는 경우 / 없는 경우 : cap이 있으면 미리 cap만큼의 양만 만들어놓고 그 이상을 crowdsale로 판매하지 않습니다. 없는 경우는 이더리움을 지급하면 지급하는 만큼 계속해서 토큰이 만들어집니다.
2. softcap(goal)이 있는 경우 / 없는 경우 : softcap이 있는 경우 기간 내에 해당 목표치를 달성하지 못하면 해당 투자액을 모두 환불하고 crowdsale을 실패한 것으로 봅니다.

일반적으로 위와 같이 진행되기는 하나, 스마트 컨트랙트 자체에는 어떤 제한도 없기 때문에, 원하는 요구사항에 맞춰 어떤 형태로든 만들 수 있습니다.