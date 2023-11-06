# [Malicious Packages] Dependency confusing attacks
[Snyk의 관련 아티클](https://snyk.io/blog/detect-prevent-dependency-confusion-attacks-npm-supply-chain-security/?_gl=1*1abu5kz*_ga*MjAyMzU2MzczMS4xNjkzODAyMDc0*_ga_X9SH3KP7B4*MTY5NzA5ODY1OS4xMC4xLjE2OTcwOTk3OTQuMC4wLjA.)을 실습한 예제입니다.
Private npm registry를 사용할 때 어떤 취약점이 발생할 수 있는지 확인해 봅니다.


## 1. Getting Started
### 1.1. Private npm registry 실행하기

로컬 환경에서 private npm registry를 띄우기 위해 오픈 소스 프로젝트 [Verdaccio](https://verdaccio.org/)를 사용합니다.
아래 명령어를 입력하여 전역으로 설치해 주세요.
```bash
$ npm install -g verdaccio
```

다음 명령어를 입력하여 private npm registry를 실행합니다.
```bash
$ verdaccio
```


### 1.2. 테스트 패키지를 로컬 Private npm registry에 등록하기
다음 명령어를 입력하여 사용자 추가 및 로그인을 진행합니다.
(가상의 ID/PW를 입력해 주세요.)

```bash
$ npm adduser --registry http://localhost:4873/ ## 입력하면, ID와 PW를 입력하라는 터미널 메시지가 출력됩니다.
```

이후 테스트 패키지의 루트 디렉토리로 이동하여, Private npm registry에 각각 Publish 해줍시다.

```bash
## 다음은 테스트용 패키지 중 `death-star-secret-hyper-matter-reactor`에 대한 예시입니다.
## 다른 테스트 패키지들도 동일한 방식으로 진행해 주세요.
$ cd packages/death-star-secret-hyper-matter-reactor
$ npm publish  --registry http://localhost:4873
```


아래와 같은 사진이 보이면 지금까지 잘 따라온 겁니다!
<img src="/docs/thumbnail-1.png" width="65%">


## 2. 보안 취약점 확인해 보기
Dependency confusing attack은 Private npm registry에 등록된 네임 스페이스를 공개 저장소에도 등록할 수 있다는 점을 이용한 공격 방식입니다.

예제에서 사용하는 패키지 중 [superlaser](https://www.npmjs.com/package/superlaser)는 악성 사용자가 공개 저장소에 등록했다고 가정합시다. (Snyk 아티클에서 제공해주는 예제 패키지입니다. :>)

여기서 악성 패키지의 버전이 사설 패키지 저장소보다 버전이 높다는 점에 주목하세요!
(`1.99999.9999` vs `1.0.0`) 

### 2.1. 패키지 최초 설치 시
아래 명령어를 입력하여 `yarn.lock`에 `superlaser`의 패키지 버전이 무엇인지 각각 확인해 봅시다.

```bash
$ yarn add superlaser ## 악성 공격자의 패키지가 설치됩니다.
$ yarn add superlaser@1.0.0 ## 사설 패지키 저장소의 패키지가 설치됩니다.
```

```yml
## yarn.lock
## Case 1. 버전을 명시하지 않고 설치한 경우
"superlaser@npm:^1.0.0":
  version: 1.99999.9999
  resolution: "superlaser@npm:1.99999.9999"
  checksum: 25f4965561689e7be5e975005b31fd755726c261adce849a76f007496763e90454fd268aaa6ba1023b64908499c8ae80d37df8e2fba6a34e95461c9027cbf70e
  languageName: node
  linkType: hard


## Case 2. 사설 저장소의 버전에 맞추어 설치한 경우
"superlaser@npm:1.0.0":
  version: 1.0.0
  resolution: "superlaser@npm:1.0.0"
  checksum: 91f9bacc6432ee11a3aa34aa0c4fdb3c0b0df5404ca2ebbca4831e3949c81fa0a5aabf8c7b48618e1f053bdcc5860c367c938d368f1a19d03475d0c0f657b3f5
  languageName: node
  linkType: hard

```
버전을 명시하지 않고 최신 버전을 설치하도록 입력하면, 공개 저장소와 사설 저장소 중 높은 버전의 패키지를 설치하게 됩니다. :0...

### 2.2. 패키지 업데이트 시 취약점

아래 명령어를 입력해서 사설 패키지 기준으로 설치된  `superlaser`를 최신 버전으로 업데이트 해봅시다.
```bash
$ yarn up superlaser
```

정말 놀랍게도 악성 패키지가 대신 설치됩니다. :0...
```yml
## yarn.lock
## 1. 업데이트 전
"superlaser@npm:1.0.0":
  version: 1.0.0
  resolution: "superlaser@npm:1.0.0"
  checksum: 91f9bacc6432ee11a3aa34aa0c4fdb3c0b0df5404ca2ebbca4831e3949c81fa0a5aabf8c7b48618e1f053bdcc5860c367c938d368f1a19d03475d0c0f657b3f5
  languageName: node
  linkType: hard


## 2. 업데이트 후
"superlaser@npm:^1.0.0":
  version: 1.99999.9999
  resolution: "superlaser@npm:1.99999.9999"
  checksum: 25f4965561689e7be5e975005b31fd755726c261adce849a76f007496763e90454fd268aaa6ba1023b64908499c8ae80d37df8e2fba6a34e95461c9027cbf70e
  languageName: node
  linkType: hard

```

##3. Reference
- [[Snyk] Malicious Packages](https://docs.snyk.io/manage-risk/find-and-manage-priority-issues/malicious-packages#types-of-malicious-packages)
- [[Snyk Blog] Detect and prevent dependency confusion attacks on npm to maintain supply chain security
](https://snyk.io/blog/detect-prevent-dependency-confusion-attacks-npm-supply-chain-security/?_gl=1*1abu5kz*_ga*MjAyMzU2MzczMS4xNjkzODAyMDc0*_ga_X9SH3KP7B4*MTY5NzA5ODY1OS4xMC4xLjE2OTcwOTk3OTQuMC4wLjA.)

- [[Github] Dependency confusion](https://github.com/x1337loser/Dependency-Confusion)

- [[Github] Confused](https://github.com/visma-prodsec/confused)