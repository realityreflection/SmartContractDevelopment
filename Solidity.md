
# Solidity

이더리움 컨트랙트를 구성하는 코드는 EVM이라는 로우 레벨, 스택 기반의 바이트 코드 언어로 작성됩니다. 따라서 EVM 바이트코드로 컴파일될 수 있는 언어라면 뭐든지 이더리움 컨트랙트를 작성하는데 사용할 수 있습니다. 여기서는 그 중 가장 기본적이고 대표적인, 이더리움에서 제공하는 solidity라는 언어를 다뤄봅니다.

Solidity에 대해 다루기 전에 우선 이더리움 프로토콜을 이루는 구성 요소들에 대해 간단히 살펴봅시다. 아래 내용들은 이더리움 백서의 내용을 기반으로 작성되었습니다. 더 정확하고 자세한 내용을 이해하고 싶다면 [이더리움 백서](https://github.com/ethereum/wiki/wiki/[Korean]-White-Paper)를 한 번 정독해 보시는 것을 추천합니다.

## 이더리움 어카운트

이더리움 어카운트는 다음 네 가지 요소를 가지고 있습니다.
- 논스(nonce) : 트랜잭션이 한 번만 처리되게 하는 카운터
- 어카운트의 현재 이더 잔고
- 어카운트의 컨트랙트 코드(있을 수도 있고 없을 수도 있음)
- 어카운트의 저장 공간

트랜잭션이 발생할 때 트랜잭션 수수료는 해당 트랜잭션을 실행하 어카운트의 이더 잔고에서 지불됩니다. 어카운트의 경우 외부 소유 어카운트(Externally Owned Accounts)와 컨트랙트 어카운트(Contract Accounts)가 있는데, 외부 소유 어카운트는 컨트랙트 코드가 존재하지 않습니다. 프라이빗 키로 관리되는 일반적인 지갑들을 생각하시면 됩니다. 컨트랙트 어카운트들은 메시지를 받을 때마다 코드를 실행하고, 내부 저장 공간에 저장된 값들을 변경하거나 다른 메시지를 보내고, 컨트랙트를 생성하는 등의 작업을 수행하게 됩니다.

## 메시지

컨트랙트는 다른 컨트랙트에게 메시지를 전달할 수 있습니다. 메시지는 아래 요소들을 가지고 있습니다.

- 메시지 발신처
- 메시지 수신처
- 메시지와 함께 전달되는 이더
- 데이터 필드(optional)
- STARTGAS 값

이 메시지들은 컨트랙트 실행 중에만 존재하는 가상의 오브젝트로, 간단하게 다른 컨트랙트를 실행하기 위해 사용되는 데이터라고 생각하시면 됩니다. 이 실행 환경 상에서 gas라는 개념이 중요한데, 이 내용은 조금 더 아래에서 같이 다루겠습니다.

## 트랜잭션

트랜잭션과 메시지는 유사한 개념인데, 메시지가 컨트랙트가 다른 컨트랙트에게 전달하는 오브젝트라면 트랜잭션은 외부 소유 어카운트(컨트랙트가 없는)가 보낼 데이터를 말합니다. 트랜잭션은 아래와 같은 내용을 포함하고 있습니다.

- 메시지 수신처
- 발신처를 확인할 수 있는 서명
- 발신처가 수신처로 보내는 이더의 양
- 데이터 필드(optional)
- STARTGAS
- GASPRICE

gas 개념은 EVM의 실행 환경이 튜링 완전하기 때문에 붙은 개념인데, 쉽게 말해 명령어 실행에 대한 수수료라고 생각하시면 됩니다. 튜링 완전하다는 말은 무한 루프를 구성할 수 있다는 얘기인데, 그렇다면 명령어 실행에 아무 제약이 없다면 악의적인 공격을 방어할 수가 없어진다는 뜻입니다. 따라서 이더리움 실행 환경에서는 기본적으로 하나의 계산 단계를 수행하기 위해 1 gas의 비용을 소모하게 해 두었습니다. 그래서 트랜잭션은 startgas(해당 컨트랙트를 수행하기 위한 기본 가스 량. 만약 컨트랙트 수행 중에 가스 소모량이 이 양을 넘어선다면 해당 트랜잭션은 실패한 트랜잭션으로 취급합니다) 및 1 gas당 비용인 gaspric(이더 기준) 값을 포함하게 됩니다. 트랜잭션은 실패하는 경우에도 소모된 양 만큼의 가스가 채굴자에게 지급됩니다.