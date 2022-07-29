---
title: "npm vs yarn"
categories:
  - package
date: 2022-07-29 06:00:25 +0900
tags:
  - package install
  - npm
  - yarn
---

포트폴리오를 웹으로 작성하기 위해 현재까지 완료된 작업은 다음과 같다.
1.  Portfolio frontend(React) 제작 및 Dockerfile 생성
2.  Nginx configuration 설정
3.  docker-compose 설정. react-web(expose 3000), nginx(80:80)
4.  AWS EC2 인스턴스 고정 IP와 AWS-route-53 도메인 연결

react docker file 설정할 때, npm으로 패키지를 install하도록 설정했는데 속도가 매우 느렸다.
그래서 대안을 찾던 와중 npm 와 yarn을 비교분석 해놓은 아래의 자료가 있었다.

<div class="notice" markdown="1">
#### Yarn vs. NPM: How to Choose?
It's essential to consider the advantages and disadvantages of both NPM and Yarn when deciding which one to use.

### [Yarn]

#### Advantages

Supports **parallel installation** and **Zero installs**, both of which dramatically increase performance.
Newer versions of Yarn offer a more secure form of version locking.
Active user community.

#### Disadvantages

Yarn doesn't work with Node.js versions older than version 5.
Yarn has shown problems when trying to install native modules.

### [NPM]

#### Advantages

Easy to use, especially for developers used to the workflow of older versions.
Local package installation is optimized to save hard drive space.
The simple UI helps reduce development time.

**Disadvantages**
The online NPM registry can become unreliable in case of performance issues. This also means that NPM requires network access to install packages from the registry.
Despite a series of improvements across different versions, there are still security vulnerabilities when installing packages.
Command output can be difficult to read.
</div>


요약하면, 속도측면에서 npm은 패키지 install을 시퀀스로 처리하는 반면, yarn은 병렬로 installing하기에 yarn이 속도측면에서 퍼포먼스가 높다. 특히 **다량의 package**를 설치할 수록 yarn이 퍼포먼스가 더 높아진다.

<img src="/assets/images/yvsn.png">

따라서 필자는 패키지를 다량 가져오기에 병렬처리가능한 yarn을 사용함으로써 cache가 없는 초기 환경일 때, 약 **5분** 내로 배포가 가능하였다 (npm took **5 hours** to install with no-cache)

|                       | Yarn / Yarn 2.0                                          | NPM                                                                |
| :-------------------: | -------------------------------------------------------- | ------------------------------------------------------------------ |
|     Zero Installs     | Uses a "**.pnp.cjs**" file to reinstall packages offline | Unsupported                                                        |
|  Usage of Workspaces  | Supported                                                | Supported                                                          |
|         Speed         | **Parallel** installation                                | **Sequential** installation                                        |
|    Remote Scripts     | Supported using the command "yarn dlx"                   | Supported using the command "npx"                                  |
|      Plug'n'Play      | Uses a ".pnp.cjs" file                                   | Unsupported                                                        |
|     License Check     | Checks each package download while installing            | Unsupported                                                        |
| Generating Lock Files | Automatically created as "yarn.lock"                     | Automatically created as "package-lock.json"                       |
|     Dependencies      | Installs using the "yarn" command                        | Installs dependencies one by one through the "npm install" command |



따라서 최종 변경된 react의 dockerfile은 아래와 같다.





<p style="margin:0 0 0 0; background-color:#f2f3f3; border-radius:0.2em; text-align:center;">/react-portfolio-website/client/Dockerfile</p>

```dockerfile
FROM node:18-alpine3.16
RUN mkdir /app
WORKDIR /app

ENV PATH /app/node_modules/.bin:$PATH 

COPY package.json /app/package.json
COPY yarn.lock /app/yarn.lock
RUN yarn install 
COPY . /app 

CMD ["/app/start.sh"]
COPY .env /app/.env

CMD ["npm", "start"]
```

`/app/start.sh`는 react의 default port를 변경시키기 위해 `.env` 파일을 생성하는 간단한 쉘 스크립트이다.

<p style="margin:0 0 0 0; background-color:#f2f3f3; border-radius:0.2em; text-align:center;">/react-portfolio-website/client/start.sh</p>

```yaml
#!/bin/sh
set -e

echo "make .env with PORT=$PORT"
echo "PORT=$PORT" > .env
```

또한 필자는 도커로 이를 관리하기에 image크기 또한 신경써야한다. image 크기를 **60%** 줄여 최적화한 다음의 issue를 확인하자.

(https://github.com/yarnpkg/berry/discussions/3201#discussioncomment-1086179)

(추가로 docker에서 `.pnp,cjs` 파일을 카피하여 Zero install또한 활용하도록 할 수 있다)






