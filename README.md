CI(Continuous Integration)/CD(Continous Delivery) 지속적인 통합 및 배포

## 새로운 서비스 제작 → git push → 알아서 통합 → 알아서 서버로 배포 → 인터넷에서 서비스 사용 가능

### 필요한 준비물

- Termius : EC2서버들을 내가 로컬에서 SSH key를 통해서 간단하게 이용하게 해주는 클라이언트 (Termius 말고도 여러가지 존재)

[[부분무료] 멀티 플랫폼 SSH 클라이언트: Termius](https://macnews.tistory.com/5728)

- Nginx : 경량 웹 서버로 여러가지 기능을 가지고 있다. 정적파일들(html, css, javascript)을 서비스 해주고 백엔드의 API gateway기능도 해준다. 
**reverse proxy를 이용해서 외부에서 내부에 제공하는 서비스를 이용할 때 proxy server를 통해서 들어오게 해주는 방식도 제공**

[Nginx란?](https://ssdragon.tistory.com/60)

[1. NGINX의 개념과 이해](https://velog.io/@jihyunhillpark/서버-배포-1.-NGINX란)

[[nginx] nginx에서 reverse proxy 사용하기(feat. 우선 순위)](https://icerabbit.tistory.com/116)

- Jenkins : 지속적인 서버 통합 서비스를 가능하게 해주는 툴 Java로 제작되었기 때문에 JDK가 필요하다

[젠킨스 (소프트웨어)](https://ko.wikipedia.org/wiki/젠킨스_(소프트웨어))

- Let’s Encrypt : nginx를 가지고 서버에서 접속을 할때 https로 접속하게 해주는 기능

[Let’s Encrypt 인증서로 NGINX SSL 설정하기](https://nginxstore.com/blog/nginx/lets-encrypt-인증서로-nginx-ssl-설정하기/)

Dokcer란?

[Docker란? What is docker? 도커 컨테이너, docker container 실행](https://www.redhat.com/ko/topics/containers/what-is-docker)

## 1. EC2를 받은 SSH키를 인증서로 이용해서 Termius에서 호스트로 연결한다!

![1](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/8cb39603-475b-4311-b993-c5d73f5edd4e)

Label —> 내가 이 호스트에 붙일 이름

Address —> 도메인 주소

SSH —> 22포트는 AWS기본으로 열어주는 포트

Username —> 해당 EC2 서버의 기본 이름 (ubuntu)

Password —> Set a Key 클릭

![2](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/ef0ae821-904b-48b3-94e0-5008043de35f)

Set a Key에서 Private key에 키값을 전부 넣던가 OR 받았던 ssh 키.pem 파일을 드래그 해서 집어넣던가 한 후 import한다.

![3](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/1295922c-844a-48c2-b249-683a2e429cda)

그 후 위와 같이 Host 생성되고 이를 더블클릭하면 해당 EC2 서버에 연결된다.

## 2. EC2에 Nginx와 Jenkins를 설치 (Not use Docker)

 Nginx를 sudo apt install nginx를 통해서 설치한 뒤 (sudo —> 권한, apt —> 우분투에서 쓰는 명령어)

cd /etc/nginx를 통해서 해당 경로로 들어가면 여기서 nginx 설정 파일들을 확인할 수 있다.

sites-available 경로를 들어가서 proxy 관련 설정 conf 파일을 생성하고 이를 sites-enabled에 복사하면 된다.

sites-available 경로에서 만든 conf 파일 설정 

![4](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/2606c95a-8b7e-4dbf-ab4d-b951f08ded8e)

[AWS EC2 nginx + letsencrypt](https://blog.namthplayground.com/27)

jenkins 설치

[Linux : Ubuntu 20.04 : Jenkins 설치 방법, 예제, 명령어](https://jjeongil.tistory.com/2018)

[[Ubuntu 20.04] 젠킨스(Jenkins) 설치 및 설정](https://hyunmin1906.tistory.com/272)

## Jenkins관리

Jenkins를 들어가게 되면 이제 내가 무엇을 빌드할 것이냐 gitlab 이냐 github냐에 따라서 여러 필요한 플러그인들이 존재한다. 이들을 다운 받아주자 왼쪽에서 Jenkins 관리를 눌러서 들어가면 아래와 같다.

![5](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/9a40d1d7-18dc-4cbb-886a-ca02ea052bca)

플러그인 관리를 들어가서 왼쪽에서 Available plugins를 클릭 한 후 필요한 플러그인들을 검색해서 설치해준다.

시스템 설정에서는 써있는 대로 환경변수 및 경로 정보들을 설정할 수 있다.

Manage Credentials를 들어가면 내가 gitlab이나 github에 관해서 접근할 때 필요한 Credentials를 확인할 수 있다.

이제 새로운 Item을 만들어서 CI/CD를 구축 해주자

Dashbaord에서 

왼쪽위에 새로운 Item을 클릭

![6](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/3fa7469d-c0aa-4b3f-b1d4-0b402a40a7ce)

Freestyle 방법이 존재하고 Pipeline 방법이 존재한다.

각각 장단점이 존재하지만

빌드, 배포중에 좀 더 세밀하게 조정하거나 확인할 것이 있으면 Pipeline으로 작성한다. (대개 이거 쓰는듯?)

그러면 이제 구성을 들어가게 된다.

## 구성에서 Build Triggers 부분을 확인하자

![7](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/28f64e00-6a15-435d-9e65-0da526776d20)

언제 빌드를 유발할지 선택할 수 있다.

그 후 고급을 클릭하자

제일 아래 부분을 보면 시크릿 토큰을 생성할 수 있다.

![8](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/46d06540-3ced-458c-88ac-dee88cf6df5b)

이 토큰을 가지고 Gitlab에서 Webhook에 등록을 해주면된다.

MainTainer만 Setting를 건들 수 있다.

![9](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/ad3d8d23-fc0d-40c0-96e3-7467daa569cd)

URL 부분엔 

위에 나와 있는 BuildTrigger에 나와있는 jenkins 의 item URL을 가져다 쓰고 Token도 가져다 쓴 후 
언제 Trigger를 유발 시킬지 설정한 후 Add Webhook을 하면 된다. 그 후 Test도 가능

## 마지막으로 Pipeline 작성

![10](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/48dad834-01fc-499c-8dc6-628c8a043330)

여기서도 방법이 2 가지 있다.

1. Pipeline script로 젠킨스에서 script를 작성하는 것이고
2.  Pipeline script from SCM을 통해서 gitlab과 credentials 통해서 연결 한 후 gitlab에 올라간 Jenkinsfile.groovy를 통해서 작성된 script를 이용하는 것이다.

2 번 방법을 사용하려다 보니 docker에 환경변수를 설정해서 api 키나 db 연결부분을 설정하는 부분이 들어가게 되는데 이것을 gitlab에 그대로 올리게 되면 gitignore한 의미가 사라져서 1 번 방법을 사용했다.

```groovy
pipeline {
  agent any
  // gitlab연동
  stages {
    stage('Checkout') {
      steps {
        git credentialsId: 'sojin', url: 'https://lab.ssafy.com/s08-final/S08P31B104.git', branch: 'dev'
      }
    }
    // docker 이미지 빌드
    stage('Vue.js Image Build') {
      steps {
        script {
          def frontendDir = "${env.WORKSPACE}/frontend/vue3pwa"
          def dockerfile = "${frontendDir}/Dockerfile"
          docker.build("birdchain-front:${env.BUILD_NUMBER}", "--build-arg VUE_APP_KAKAOMAP_KEY=ce58167b147abd28615c0b0b9990cd57 -f ${dockerfile} ${frontendDir}")
        }
      }
    }
    // 중복을 피하기 위한 컨테이너 정지 후 삭제
    stage('Remove Container') {
      steps {
        script {
          try {
            sh 'docker ps -f name=frontend -q | xargs --no-run-if-empty docker container stop'
            sh 'docker container ls -a -f name=frontend -q | xargs -r docker container rm'
          } catch (err) {
            echo "Failed to stop the container"
          }
        }
      }
    }
    // 새로 만든 이미지로 컨테이너 run
    stage('Run Container') {
      steps {
        script {
          docker.image("birdchain-front:${env.BUILD_NUMBER}").run("--name frontend -p 3000:80")
        }
      }
    }
  }
}
```

## Pipeline 설명

stages로 한 후 stage로 각각 stage를 실행한다.여기선 실행되는 script의 이름을 내가 지어준다.

steps를 통해서 내가 어떤것을 실행할지 여기서 구현한다.

Checkout은 내가 미리 등록해놓은 credentials를 통해서 credentialsID 이름을 사용한다. 그 후 url에 gitlab에서 clone해서 가져온 https 주소를 넣고 내가 Jenkins와 연동할 branch를 설정한다.

Vue이미지를 docker로 빌드해준다.

${env.WORKSPACE}는 jenkins가 깔린 기본 directory로 설정이 되어 있다. —> var/lib/jenkins/workspace (EC2에서 cd로 들어가보면 존재 확인 가능)

—build-arg를 통해서 내가 원하는 환경변수를 docker컨테이너 안에서 설정할 수 있다.

-f —>백그라운드에서도 돌아가게 하기

마지막 부분 —>var/lib/jenkins/workspace/item이름(frontend)
여기 까지 들어오면 gitlab에서 가져온 브랜치의 디렉토리 부분이고 여기서 프론트엔드 부분을 들어가서 Dockerfile을 가져다 사용해야 함으로 Dockerfile 위치까지 내가 지정해준다.

그 후 frontend라고 이름이 지어진 docker 컨테이너가 있으면 이를 멈추고 삭제를 시킨다. (똑같은 이름으로 docer 컨테이너를 run하면 중복이 됨으로)

마지막으로 docker 컨테이너에 이름을 주고 docker컨테이너와 EC2의 포트 바인딩을 설정해주면서 아까 만들었던 docker image를 run해준다.

[Docker 명령어 정리](https://captcha.tistory.com/49)

[[Jenkins] Pipeline Syntax (젠킨스 파이프라인 문법)](https://waspro.tistory.com/554)

[젠킨스 파이프라인 문법(Pipeline Syntax) 총정리](https://blog.voidmainvoid.net/104)

왼쪽 아래에는 이 item이 빌드된 횟수가 나오고 있다. 저것을 클릭하면 console.out을 통해서 에러등을 확인할 수 있다.

![11](https://github.com/Croon00/MSA_Study-inflearn-/assets/73871364/1523fb25-9788-4cf9-b5ef-5453bcbe34e2)

## 도커 이미지를 어떤식으로 빌드할지는 각 프로젝트에서 최상단 위치에 Dockerfile 이란 이름으로 생성한 후 작성해놓으면 된다.

ex) Vue프론트 Dockerfile

```docker
# 1단계: 빌드용 이미지
FROM node:18.16.0-alpine AS build

# 작업 디렉토리 설정
WORKDIR /app_front

# 환경변수 추가
ARG VUE_APP_KAKAOMAP_KEY
ENV VUE_APP_KAKAOMAP_KEY=$VUE_APP_KAKAOMAP_KEY

# package.json과 package-lock.json 복사
COPY package*.json ./

# 종속성 설치
RUN npm install

# 프로젝트 파일 복사
COPY . .

# 프로젝트 빌드
RUN npm run build

# 2단계: 실행용 이미지
FROM nginx:stable-alpine

# 빌드 결과물을 Nginx로 복사
COPY --from=build /app_front/dist /usr/share/nginx/html

# Nginx 포트 설정
EXPOSE 80

# Nginx 실행
CMD ["nginx", "-g", "daemon off;"]
```

## Nginx.conf

```jsx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

- nginx의 설정을 하는 방법은 리눅스 가상환경에 직접 설정파일을 두는 방법도 있다
- /etc/nginx/ 위치등.
- 그러나 도커 안에서 실행할때 nginx로 실행을 해야 하는 경우 가상 환경 안에 도커 환경 안에서 nginx 설정 파일을 복사해야 하는데 이것을 nginx.conf 파일을 만든 후
- # 빌드 결과물을 Nginx로 복사
COPY --from=build /app_front/dist /usr/share/nginx/html 이것을 이용하는 방법도 있따.

SpringBoot dockerfile

```docker
FROM adoptopenjdk:11-jre-hotspot
WORKDIR /app
COPY build/libs/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

springboot는 jar 파일을 배포 해준다.

jar 파일이란?

[JAR (파일 포맷)](https://ko.wikipedia.org/wiki/JAR_(파일_포맷))

## Springboot를 Docker로 생성하고 Mysql도 Docker로 생성한 경우

## 이 두개의 컨테이너는 서로 연결되어 있어야 하는데 이를 이용하려면

## Docker network를 같은 곳으로 하여 생성을 하여야 한다.

[Docker 네트워크 사용법](https://www.daleseo.com/docker-networks/)
