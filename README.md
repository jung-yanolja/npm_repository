# private npm repository

## Requirement
* verdaccio
* lerna 

> 이 프로젝트는 패키지 관리 예제 프로젝트로 lerna 를 셋팅한 저장소이다.
> 이 프로젝트를 clone 할 필요없이 직접 lerna install 해서 사용해도 된다.   
> 결과적으로 private npm 서버에 업로드되는 패키지들이 모여있는 저장소이다.   
> 전체적인 테스트를 위해서는 로컬 환경에서 verdaccio 셋팅이 필요하다. 

## verdaccio[https://github.com/verdaccio/verdaccio]
### About
* private npm 서버를 구축하기 위한 라이브러리

### install
```sh
$ npm install --global verdaccio 
```

### Run
```sh
$ verdaccio 
```
- http://localhost:4873 접속

### Getting Started
* 사용자 계정 생성
```sh
$ npm adduser --registry http://localhost:4873
```

* 일부 패키지 파일 변경    
```
$ cd packages/component1
$ echo "console.log('Hello world')" > index.js
```

* 변경사항 commit
```
$ git add *
$ git commit -m "update, file"
```

* 패키지 업로드
```sh
$ npm publish --registry http://localhost:4873
```

* 패키지 다운로드
```sh
$ npm install --registry http://localhost:4873 component1
```

디폴트 값은 access 는 누구나 접속 가능하고, publish 는 사용자 등록이 되어야 가능하다.   
아래 파일을 참고.

### Default Config File
로컬에 verdaccio 를 설치하면 아래와 같은 경로에 설정 파일이 생성된다.
```
$ vi ~/.config/verdaccio/config.yaml
```
config.yaml
```
storage: ./storage
auth:
  htpasswd:
    file: ./htpasswd
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
packages:
  '@*/*':
    access: $all
    publish: $authenticated
    proxy: npmjs
  '**':
    proxy: npmjs
logs:
  - {type: stdout, format: pretty, level: http}
```

* auth - 사용자 인증 정보
* uplinks - 써드파티 경로 ex) private npm 서버에 패키지가 없으면 npm.js 서버를 탄다.
* packages - publish 할 대상 패키지들에 대한 설정

### Authentication
verdaccio-htpasswd[https://github.com/verdaccio/verdaccio-htpasswd]   
인증된 사용자만 access, publish, unpublish 와 같은 명령의 수행이 필요할 경우.

#### How
- 패키지 권한을 변경해줘야한다.
- npm adduser 명령어를 disabled 해야한다. (누구나 npm adduser 를 실행할 수 있기에, 인증은 어렵지 않기 때문)

#### 1. 패키지 권한을 변경. => config.yaml 수정
config.yaml
```
packages:
  '@*/*':
    access: $authenticated
    publish: $authenticated
    proxy: npmjs
```
    
#### 2. npm adduser 을 통한 사용자 등록을 막는다. => config.yaml 수정   

verdaccio 인증 동작 원리
* 각 로컬에서 npm adduser 명령어를 통해 인증을 시도한다.  
* 관련 데이터는 config.yaml 에서 auth => htpasswd =>  file 에 지정된 경로로 사용자 정보들이 저장된다. 

이를 활용해서 다음과 같이 수행 가능하다.

config.yaml
```
auth:
  htpasswd:
    file: ./htpasswd
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    max_users: -1
``` 

그 후, 실제 private npm 서버에서 수동으로 htpasswd 파일에 사용자 정보를 넣어준다.
htpasswd 에 htpasswd 형태로 사용자 정보를 넣어주기만 하면 되므로, 이를 활용해 자동화 가능.
[http://www.htaccesstools.com/htpasswd-generator/]

npm adduser 를 통해 사용자 등록 시도할 경우 409 Conflict 발생.
결과적으로 인증되지 않은 사용자는 배포된 패키지조차 볼 수 없고, publish 을 불가능하게 할 수 있다.

* 사용자 등록
```sh
$ npm adduser --registry http://localhost:4873
```

* 현재 로그인된 사용자 조회
```sh
$ npm whoami --registry http://localhost:4873
```

* 사용자 로그인
```sh
$ npm login --registry http://localhost:4873
```

* 사용자 로그아웃
```sh
$ npm logout --registry http://localhost:4873
```

## lerna[https://github.com/lerna/lerna]
### About
* 많은 패키지를 하나의 저장소로 관리하는 기법 monorepo 를 지원하는 라이브러리
* publish 와 동시에 자동 버젼 관리 

### install
```sh
$ mkdir lerna-repo 
& cd lerna-repo
$ npx lerna init
```

디폴트는 --independent 모드가 아님.  
버젼이 독립적이지 않고 모든 패키지에 한번에 적용된다.  
필요시 `lerna init --independent` 실행 or lerna.json 에서 { "version": "independent" } 수정

### Getting Started

* lerna.json
```json
    {
      "version": "independent",
      "npmClient": "npm",
      "command": {
        "publish": {
          "ignoreChanges": [
            "ignored-file",
            "*.md"
          ],
          "message": "chore(release): publish",
          "registry": "{URL}"
        },
        "bootstrap": {
          "ignore": "component-*",
          "npmClientArgs": [
            "--no-package-lock"
          ]
        }
      },
      "packages": [
        "packages/*"
      ]
    }
```

패키지 배포
```
$ lerna publish
```
명령어 실행 git push 까지 반영된다. 

"registry" 값을 실제 사용할 npm 서버 경로로 수정 필요