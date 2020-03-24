# private npm repository

## verdaccio[https://github.com/verdaccio/verdaccio]
### About
* private npm 서버를 구축하기 위한 라이브러리

## install
```sh
$ npm install --global verdaccio 
```

#### Run
```sh
$ verdaccio 
```

## Getting Started
사용자 계정 생성
```sh
$ npm adduser --registry http://localhost:4873
```
패키지 업로드
```sh
$ npm publish --registry http://localhost:4873
```

#### Default Config File
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

## Getting Started

### lerna.json
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

"registry" 값을 실제 사용할 npm 서버 경로로 수정 필요