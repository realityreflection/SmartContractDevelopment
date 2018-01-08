
# 개발 환경 세팅

개발 환경은 여러 가지 방법으로 세팅할 수 있는데, 여기서는 python을 이용한 스마트 컨트랙트 개발을 기준으로 설명합니다.

python을 사용한 일반적인 스마트 컨트랙트의 개발 환경은 다음과 같습니다.

- OSX or linux
- py-solc
- populus

py-solc는 solidity 컴파일러인 solc를 파이썬 환경에서 쓸 수 있게 만들어 둔 것입니다. populus는 만든 컨트랙트를 배포하기 전 테스트 환경을 제공해주는 프레임워크입니다. 아쉽게도 populus가 윈도우즈 환경을 지원하지 않아 OSX 혹은 linux환경을 사용해야 합니다. 윈도우즈 사용자의 경우 Virtual Box 등을 이용하여 리눅스 환경을 구성하면 됩니다.

이 문서는 전부 linux(ubuntu) 환경을 기준으로 설명합니다. python 및 pip등의 도구는 이미 설치했다고 가정합니다.

## py-solc

사용법 및 자세한 설명은 [py-solc](https://github.com/ethereum/py-solc)를 참고하시면 됩니다. 설치는 아래 커맨드를 실행하기만 하면 됩니다.

```
pip3 install py-solc
```

## populus

populus는 아래 커맨드를 실행해서 설치할 수 있습니다.

```
sudo apt-get install libssl-dev
pip3 install populus
```

populus는 python을 통해 블록체인 환경을 쉽게 테스트할 수 있게 해줍니다. 


### populus project setting

우선 populus를 이용해 프로젝트를 초기화해 봅시다. 프로젝트를 생성하고 싶은 폴더에서 터미널을 생성한 뒤 아래의 커맨드를 입력합니다.

```
populus init
```

프로젝트를 초기화하고 나면 `./contracts` 폴더 및 `./tests` 폴더가 생성되고 그 안에 제일 기본적인 solidity 파일 및 테스트 케이스가 미리 작성되어 있습니다. 문서에는 프로젝트 기본 세팅 파일인 `./populus.json` 파일이 자동으로 만들어진다고 되어 있는데 해당 파일이 생성이 안 되는 경우가 있습니다. 기본적으로 생성이 되지 않았을 경우 populus.json 파일에 아래 내용을 작성해서 저장합니다.

```json
{
    "version" : "7",
    "compilation" :
    {
        "contracts_source_dirs" : ["./contracts"],
        "import_remmapings" : []
    }
}
```

이제 한 번 실제로 solidity 파일을 컴파일하고 테스트해봅시다.

```
populus compile
py.test
```

둘 모두 정상적으로 동작한다면 개발 환경 세팅이 잘 완료된 것입니다.

### solidity coding

solidity의 syntax highlighting을 지원하는 편집기라면 뭘 쓰든 상관없습니다.

추천 : visual studio code