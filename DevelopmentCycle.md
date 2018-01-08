
# Development Cycle

컨트랙트를 python, populus, web3.py를 이용해 개발하는 과정에 대해 알아봅시다.

일반적으로 컨트랙트의 개발 과정은 다음과 같은 순서를 통해 이루어집니다.

- 컨트랙트 코드 작성
- 컨트랙트 코드 테스트, 버그 수정
- 로컬 체인에 컨트랙트 배포, 동작 확인
- mainnet(이더리움 블록 체인)에 배포
- 블록체인 상의 컨트랙트와 상호작용하기(클라이언트 사이드 코드 구성)

## 컨트랙트 코드 작성/ 테스트

solidity를 이용해 코드를 작성하는 방법은 앞서 solidity 파트에서 다뤘습니다. 이렇게 코드를 작성하고 나면, `populus compile` 커맨드를 통해 컨트랙트를 컴파일할 수 있습니다.

만들어진 컨트랙트 코드가 정상적으로 동작하는지는 주로 pytest를 이용한 유닛 테스트로 검증하게 됩니다. 이 테스트에서 쓰이는 체인은 기본적으로 매 실행마다 초기화되는 테스트용 로컬 체인이기 때문에 부담없이 테스트 케이스를 작성하면 됩니다. pytest의 사용방법에 대한 설명은 생략하겠습니다.

간단하게 Ownable 컨트랙트를 기반으로 테스트가 어떻게 구성될 수 있는지 알아봅시다. 자세한 내용은 [populus 문서](http://populus.readthedocs.io/en/latest/dev_cycle.part-03.html)를 확인해보시면 됩니다.

### solidity 파일

앞서 작성한 Ownable 컨트랙트의 소스 코드입니다(예시).

```javascript
pragma solidity ^0.4.18;

contract Ownable {
    address public owner;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    function Ownable() public {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0));
        OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }
}
```

### python 테스트 스크립트

아래와 같이 테스트 스크립트를 작성할 수 있습니다.

```python
import pytest
import web3.contract
from ethereum.tester import TransactionFailed

@pytest.fixture
def token_owner(accounts):
    return accounts[0]

@pytest.fixture
def ownable_token(chain, token_owner):
    args = []

    transaction = {
        'from' : token_owner
    }

    #deploy할 컨트랙트의 이름, 해당 컨트랙트 생성자의 인자, 실행될 트랜잭션 정보를 넘겨주면
    # 그 결과 컨트랙트 어카운트의 주소와 해당 트랜잭션의 해쉬값이 리턴된다.
    token, tx_hash = chain.provider.deploy_contract('Ownable', deploy_args=args, deploy_transaction=transaction)

    return token

def test_ownable(ownable_token, token_owner):
    assert ownable_token.call().owner() == token_owner # call()은 실제 트랜잭션을 발생시키지 않고 내부 값만 확인할 때 사용

def test_change_owner(ownable_token, token_owner, accounts):
    ownable_token.transact({'from':token_owner}).transferOwnership(accounts[1])

    assert ownable_token.call().owner() == accounts[1]

def test_change_owner_fail(ownable_token, token_owner, accounts):
    with pytest.raises(TransactionFailed): # transaction이 실패해야만 하는 상황에 대한 테스트는 이걸로 수행
        ownable_token.transact({'from':accounts[1]).transferOwnership(accounts[2])
    
    assert ownable_token.call().owner() == token_owner
```

## 로컬 체인에 배포

실제 체인에 바로 배포해서 테스트하는 건 돈(gas 지불 비용)이 들기 때문에 좋지 않습니다. 실제 체인과 완전 동일한 로컬 체인에 먼저 배포해서 잘 동작하는지 확인한 후 실제 체인에 배포해야 합니다. 컨트랙트 코드는 한 번 배포되면 절대 수정할 수 없기 때문에(migration을 통해 수정할 수 있지만, 번거롭고 migration과정에서 지불되는 가스만큼 돈이 든다) 최대한 철저하게 테스트를 해봐야 한다는 점을 명심해야 합니다.

### 로컬 체인 만들기

터미널에서 아래 커맨드를 수행하면 새로운 로컬 체인을 생성할 수 있습니다. 이 로컬 체인은 `./chains`폴더 아래에 저장됩니다.

```
populus chain new (chain-name)
```

체인을 생성한 후 아래 스크립트들을 실행하면 해당 체인 환경이 실행됩니다.

```
populus chain new local
chains/local/./init_chain.sh
chains/local/./run_chain.sh
```

### 배포

이렇게 체인을 실행한 후 다른 터미널을 켜서 아래 커맨드를 실행하면 컨트랙트를 배포할 수 있습니다.

```
deploy --chain local TestContract
```

다만 해당 컨트랙트가 뭔가 인자를 요구로 하는 경우 단순히 저렇게 deploy가 되지는 않습니다. 복잡한 처리가 필요한 경우 web3를 이용한 스크립트를 작성해서 테스트해봐야 합니다.

### test on local chain using web3

우선 만든 체인을 populus 프로젝트에서 쓸 수 있도록 프로젝트 설정 파일에 새로 만들어진 체인의 정보를 추가해줘야 합니. 다음과 같이 프로젝트 설정 파일을 수정해봅시다.

```json
{
    "version" : "7",
    "compilation" : {
        "contracts_source_dirs" : [ "./contracts" ],
        "import_remappings" : []
    },
    "chains" : {
        "local" : {
            "chain" : {
                "class" : "populus.chain.ExternalChain"
            },
            "web3" : {
                "provider" : {
                    "class" : "web3.providers.ipc.IPCProvider",
                    "settings" : {
                        "ipc_path" : "./chains/local/chain_data/geth.ipc"
                    }
                }
            },
            "contracts" : {
                "backends" : {
                    "JSONFile" : { "$ref" : "contracts.backends.JSONFile" },
                    "ProjectContracts" : {
                        "$ref" : "contracts.backends.ProjectContracts"
                    }
                }
            }
        }
    }
}
```

새로 만든 local chain이 어떤 종류의 체인이고, 컨트랙트를 어떻게 쓸지, web3로 통신을 어떻게 할지 등등의 설정이 추가되어야 사용할 수 있습니다. 이렇게 설정 파일에 체인 관련 정보를 추가하고 나면 아래와 같이 스크립트를 작성해서 로컬 체인에 배포하고 테스트해 볼 수 있습니다.

```python
from populus.project import Project

proj = Project(project_dir='./')

with proj.get_chain('local') as chain:
    args = []

    owner = chain.web3.eth.coinbase

    transaction = {
        'from' : owner
    }

    contract, _ = chain.provider.get_or_deploy_contract('Ownable', deploy_args=args, deploy_transaction=transaction)

print(contract.call().owner())
print(contract.address)
```

`get_or_deploy`를 이용하면 이미 이전에 해당 컨트랙트를 체인에 배포한 적이 있는 경우 새로 배포하지 않고 기존에 배포된 컨트랙트의 주소를 리턴해줍니다. 그 외의 부분은 이전에 pytest를 통해 작성했을 때와 동일하기 때문에 똑같은 방식으로 여러 가지 기능이 로컬 체인에 배포된 상태에서도 잘 동작하는 지 확인해보면 됩니다.