# 1. Docker란?
컨테이너 기반의 오픈소스 가상화 플랫폼

어떤 프로그램을 외부 환경과 격리시켜 구동할 수 있게 해주는 소프트웨어

OS환경에 크게 영향을 받지 않게끔 한다고 생각하면 됨

## Container란
OS 상에 논리적인 영역(컨테이너)을  구축하고, app이 작동하는데 필요한 요소들을 모아 별도의 서버처럼 동작하는 것 
infrastructure: lam, cpu 등
Host Operation System: 운영체제
Docker
App들 
<img src="../img폴더/docker/Docker%20whole%20image.png"/>

Docker라는 계층을 만들어서, app 들을 올리는 기술들을 Docker 라고 함

어디에 겹쳐진게 아니라, container처럼 표현하게 됨, 격리되어 있는 환경들이라고 말을 많이 함, 


a,b 환경이 아예 달라도 영향을 주지 않는다는 것, window 프로그램 설치를 많이 쓸텐데, 이런 소프트 웨어 사용시

시스템 요구사항을 보고 설치하게 되는데, lam, hard등 시스템 요구사항을 보고 설치하게 되는데, 이런것들이 사실 굉장히 까다롭다.

docker는 기본적으로 linux 기반으로 동작해서 Linux 기반으로 사용한다는 것이라고 생각하면 될듯

linux app을 사용한다고 생각하면 편할듯, 

장점으로는 필요한 요소만으로 구성되어 있어 오버헤드가 적다는 것이 장점이다. docker 가 나왔을때 vm과 비교가 많이 됐는데 vm을 각각 이미지를 설치해서 사용했었는데

운영체제를 설치한다는 의미이고, 운영체제의 중요 기술을 일부분 가져와서 os로 보이게끔 세팅해서 보여준다고 생각하면 된다.

쓸때 없는 기능들을 줄였다는 것이 장점이라는 것


## Docker 기초 강의
Docker 사용에 초점을 맞춰 진행할 예정

가상화 기술과 같이 설명이 어려운 부분은 최대한 이해하기 쉽게 풀어 설명할 예정, 이 과정에서 구체적인 기술들을 설명하는 것은 생략될 수 있음

구체적이고 어려운 내용들은 이후 과정에서 다룰 예정

간혹 사용 소프트웨어가 아닌 직접 개발한 app을 가동해보는 시간을 가질 수도 있음, 이때는 스프링 부트 강의에서 사용하고 있는 프로젝트를 사용할 예정

# 2. 도커 컨테이너 구조 및 커맨드 사용법
도커 컨테이너는 컨테이너 레이어와, 이미지 레이어로 구성되어 있음 

컨테이너를 만들기 위해서는 이미지라는 것이 필요한데 이미지는, 도커 허브, 여러가지 레포지토리에서 가져올 수 있는 형태를 말함, 

이미지 레이어라는 몇가지 층으로 되어 있는데  읽기 전용의 계층이다로 보면 될거같다.

이미지를 컨테이너에 올리게 되면, 하나의 계층이 추가되고, 이를 컨테이너 레이어라고 부르며, 빌드와 쓰기가 가능해짐 사용하면서 추가되는 변경 사항은

컨테이너 레이어에 추가가된다는 것이다. 

다시 정리하자면 컨테이너 레이어는 읽기/쓰기가 모두 가능한 계층으로 최상단 레이어에 추가된다. 또한 이후 컨테이너를 실행하고 진행되는 변경 사항은 이 계층에 저장된다. 

이미지 레이어는 읽기 전용 계층으로 다른 컨테이너와 공유할 수 있는 레이어를 말한다. 

ex) 리눅스, ubuntu를 컨테이너로 사용한다 하면
1. 기본적으로 필요한 이미지 레이어를 내려받게 된다. 
   
여러 컨테이너로 가용한다 하면, 이미지레이어는 공유하지만, 각각 컨테이너 마다 컨테이너 레이어를 하나씩 붙여주며, 공유하지 않는 계층으로 이해하면 된다.

## 도커 명령어
도커를 사용하면서 컨테이너도 사용하고, 이미지, 볼륨, 네트워크 같은 것들이 있다. 여러 컴포넌트들이 존재하는데 제대로 사용하기 위해서는 명령어가 필요하다. 

```sh
docker {대상} {커맨드} {옵션} {인자}

커맨드 대상: container, image, volume, nerwork 
```

도커에서 사용할 수 있는 커맨드 리스트는 다음과 같다.
1. 'docker'입력
2. 'docker [command 대상]' --help 입력
3. 위와 같은 방법으로 커맨드 수준을 높이고 뒤에 --help 를 입력하면 됨

### docker container 이후에 작성하는 주요 커맨드는 아래와 같음

|커맨드|설명|주요옵션
|:---:|:---:|:---:|
|start|컨테이너 실행|-i|
|stop|컨테이너 정지||
|create|컨테이너 생성|--name, -e, -p, -v|
|run|이미지 내려받고 컨테이너를 생성 및 실행|--name, -e, -p, -v,-d,-i,-t|
|rm|컨테이너 삭제|-f, -v|
|exec|컨테이너에서 프로그램 실행|-i, -t|
|ls|컨테이너 목록 출력|-a|
|cp|컨테이너와 호스트간 파일 복사|-i|
|commit|컨테이너를 이미지로 변환|-i|

### docker image 이후에 작성하는 주요 커맨드는 다음과 같음
|커맨드|설명|주요옵션
|:---:|:---:|:---:|
|pull|이미지를 내려받음||
|rm|이미지 삭제||
|ls|가지고 있는 이미지 목록을 출력||
|build|이미지 생성|-t|

## 주로 사용하는 옵션에 대한 설명은 다음과 같음

| 옵션 | 설명|
|:---:|:---:|
|--name| 컨테이너 이름|
|-p|포트 번호 지정|
|-v | 볼륨 설정|
|-e|환경변수 설정|
|-d|백그라운드 실행|
|-i|컨테이너에 터미널 연결|
|-t|특수 키를 사용가능하게 설정|

# 3. 도커 컨테이너와 통신하기
도커 컨테이너는 기본적으로 독립적인 환경에서 실행되기 떄문에 컨테이너 밖에서 접근이 불가능함

컨테이너와 통신하기 위해서는 컨테이너를 가동시키면서 'p' 옵션을 사용하여 호스트의 포트와 컨테이너의 포트를 설정해야 함

`-p ${host_port}:${container_port}`
이 설정을 사용하기 위해서는 호스트(서버또는 PC)에서 사용중인 포트와 번호가 겹치지 않는지 확인이 필요함


도커 예제 코드
```sh
docker run --name test1 -d httpd

docker run --name test2 -d -p 8080:80 httpd
```
--nmae test1 : test1 이라는 이름으로 컨테이너를 생성합니다.
-d : 백그라운드로 동작합니다. 
-p 8080:80 > 호스트의 포트는 8080, 컨테이너의 포트는 80으로 세팅하여 네트워크를 설정합니다.

앞으 커맨드를 실행한 후에 docker 데스크 탑으로 컨테이너 상태를 확인 또는 아래 커맨드를 입력하여 상태를 확인
```sh
docker ps -a
docker container ls -a
```
`-a` 가 없으면 실행중인 컨테이너만 보여주게 됨 a라는 옵션을 붙여주면, 다양한 스테이터스를 가지고 있는 것들을 보여준다는 의미

컨테이너 실습을 마치면 아래 커맨드를 통해 실행을 중지하고 삭제하는 작업을 수행하는 것이 좋다.
```sh
docker stop test1
docker rm test1
```

# docker image download
```sh
docker pull httpd
```

# 4. 도커 컨테이너와 통신하기
```sh
docker run --name test1 -d httpd
docker run --name test1 -d -p 8080:80 httpd
```
바로 하면 오류가 남

삭제하려면 먼저 정지하고 해야 한다는 것  기억하기 


# 5. 도커파일 작성하기 
도커파일은 도커 이미지를 생성하기 위한 스크립트 파일, 여러 키워드를 사용해 dockerfile을 작성해 빌드를 보다 쉽게 수행이 가능함

주요 인스트럭션(키워드)은 다음과 같다.
- From 
  base가 되는 이미지를 지정, 주로 OS 이미지나, 런타임 이미지를 지정함
- Run
  이미지를 빌드할때 사용하는 커맨드를 설정할때 사용 
- ADD
  이미지에 호스트의 파일이나 폴더를 추가하기 위해 사용, 만약 이미지에 복사하려는 디렉토리가 존재하지 않으면 docker가 자동으로 생성
- COPY
  호스트 환경의 파일이나 폴더를 이미지 안으로 복사하기 위해 사용
  'ADD'와 동일하게 동작하지만 가장 확실한 차이점은 URL을 지정하거나, 압축파일을 자동으로 풀지 않음
- EXPOSE
  이미지가 통신에 사용할 포트를 지정할때 사용
- ENV
  환경 변수를 지정할떄 사용, 여기서 설정한 변수는 $name, ${name}의 형태로 사용할 수 있음, 추가로 아래와 같은 문법을 사용하여 사용할 수도 있음, **-${name:-else}:name이 정의가 안되어 있다면 'else' 가 사용됨**
- CMD
  도커 컨테이너가 실행될떄 실행할 커맨드를 지정, RUN과 비슷하지만 CMD는 도커 이미지를 빌드할떄 실행되는 것이 아니라 컨테이너를 시작할때 실행된다는 것이 다름
- ENTRYPOINT
  도커 이미지가 실행될떄 사용되는 기본 커맨드를 지정(강제)
- WORKDIR
  RUN, CMD, ENTRYPOINT등을 사용한 커맨드를 실행하는 디렉토리를 지정 -w 옵션으로 오버라이딩 할 수 있음
- VOLUME
  영구적인 데이터를 저장할 경로를 지정할떄 사용, 호스트의 디렉토리를 도커 컨테이너에 연결, 주로 휘발성으로 사용되면 안되는 데이터를 저장할 떄 사용 
- etc
  SHELL, LABEL, USER, ARG, STOPSIGNAL, HEALTHCHECK

## 도커빌드
도커 파일을 실행하기 위해서는 `docker build` 커맨드를 사용
```sh
docker build ${option} ${dockerfile directory}
ex) docker build -t -test .
```
생성된 이미지를 컨테이너로 실행하기 위해서는 run 커맨드를 사용
```sh
ex) docker run --name test_app -p 80:80 test
```

#  6. 도커파일 작성하기 -실습-
docker 폴더를 만들고 그 안에 dockerfile이라는 파일을 하나 만들고, 다음으로 index.html 을 만듭니다.

```dockerfile
FROM httpd

COPY index.html /usr/local/apache2/htdocs/
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Test Page</h1>
    <p>Docker test page</p>
</body>
</html>
```
이런식으로 만든 후에 저 경로로 이동하여 (본인의 경우 `~/Documents/docker`로 이동)

`dockerr build -t test123 .` 의 명령어를 작성해 줍니다. 

명령어를 작성하면 다음과 같은 쉘이 뜨게 됩니다. 

<img src="../img폴더/docker/docker%20build.png">

이렇게 뜨고 나면 이미지 생성이 완료되었다는 뜻이며, 이를 확인 하기 위해서는

`docker image ls`를 사용하면 생성한 이미지가 뜨게 됩니다. 

기본적인 base image로 아파치의 기능을 가져오고 레이어를 얹었다고 할 수 있음

이는 `docker inspect [img 이름]` 로 알 수 있는데(docker inspect test123)

```sh
test123
"Layers": [
                "sha256:f4e4d9391e1332a5b50063a92ccc434b92fccd14962fd9c37711d864d587d98a",
                "sha256:f6d38e63f28a7e33d64976ea26d393f6c5affcd33b3aeff91e579697f8c682c7",
                "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef",
                "sha256:ad163d90e2280e79224733a70fad8cfb21d42c177bb8fd4bc6c34fbfd65f6364",
                "sha256:e6cadab87ec42fde87f753812f74502c25cdeb41e6a4964eae3113f728a8fc3c",
                "sha256:13125f10bb86a88bcb1e85e4281ab0798d821823555ca84a66b55de28312c7b8",
                "sha256:13a44ec18d4bc8e88117da7ac7bb99d3716e510efd63c101faa9b4ff4472d5a8"
            ]
        },
httpd
"Layers": [
                "sha256:f4e4d9391e1332a5b50063a92ccc434b92fccd14962fd9c37711d864d587d98a",
                "sha256:f6d38e63f28a7e33d64976ea26d393f6c5affcd33b3aeff91e579697f8c682c7",
                "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef",
                "sha256:ad163d90e2280e79224733a70fad8cfb21d42c177bb8fd4bc6c34fbfd65f6364",
                "sha256:e6cadab87ec42fde87f753812f74502c25cdeb41e6a4964eae3113f728a8fc3c",
                "sha256:13125f10bb86a88bcb1e85e4281ab0798d821823555ca84a66b55de28312c7b8"
            ]
```
이렇게 httpd는 6개, test123은 7개가 뜨는 것을 알 수 있다.  

기본 이미지 내용은 가져오고, Layer 만큼 변경사항이 있다는 것을 알 수 있다는 것

# 7. 도커 컴포즈 작성하기 
도커 컴포즈 파일은 도커 애플리케이션의 서비스, 네트워크 볼륭등의 설정을 yaml 형식으로 작성하는 파일

아래 코드는 공식 사이트 예제 이다.

```yaml
services:
    frontend:
        image: awesome/webapp
        ports:
            - "443:8043"
        networks:
            - front-tier
            - back-tier
        configs:
            - httpd-config
        secrets:
            - server-certificate
    backend:
        image: awesome/database
        volumes:
            - db-data:/etc/data
        networks:
            - back-tier
volumes:
    db-data:
        driver: flocker
        driver_opts:
            size: "10GIB"
configs:
    httpd-config:
        external: true

secrets:
    server-certificate:
        external: true

networks:
    front-tier: {}
    back-tier: {}
```
큰 틀에서 구성 요소는 아래와 같음 
- version
- services
- network
- volume
- config
- secret

이 중에 version은 deprecated 되어 더 이상 설정하지 않아도 됨

1. services는 여러 컨테이너를 정의하는데 사용됨
```yaml
services:
    frontend:
        image: awesome/webapp
        ports:
            - "443:8043"
        networks:
            - front-tier
            - back-tier
        configs:
            - httpd-config
        secrets:
            - server-certificate
    backend:
        image: awesome/database
        volumes:
            - db-data:/etc/data
        networks:
            - back-tier
```
이렇게 작성되어 있다면 `frontend`, `backend`는 각 컨테이너를 정의하고, 각 컨테이너의 이름이 됨 

컨테이너를 설정할때 사용되는 키워드는 아래와 같다.
|image|컨테이너의 이미지를 정의
|:---:|:---:|
|image|컨테이너 이미지를 정의|
|build|위 'image' 를 활용하는 방식이 아닌 dockerfile의 경로를 지정해 빌드하여 사용하는 방법|
|dockerfile|빌드할 dockerfile의 이름이 "Dockerfile" 이 아닌 경우 이름을 지정하기 위해 사용|
|ports|호스트와 컨테이너의 포트 바인딩 설정에 사용됨|
|volumes|호스트의 지정된 경로로 컨테이너의 볼륨을 마운트 하도록 설정|
|container_name|컨테이너 이름을 설정|
|command|컨테이너가 실행된 후  쉘에서 실행시킬 쉘 명령어 설정|
|environment|환경변수를 설정|
|env_file|'environment'와 동일한 기능을 수행하지만 . 이키워드를 사용하면 env 파일을 이용해서 적용이 가능함|
|depends_on|다른 컨테이너와 의존 관계를 설정|
|restart|컨테이너의 재시작과 관련하여 설정|
2. 작성된 docker.yaml 파일
   작성된 docker-compose.yml 파일을 실행하기 위해서는 아래와 같은 커맨드를 사용
   
   `docker-compose up`
   
   추가로 아래와 같은 주요 옵션을 사용할 수 있습니다. 

   - -f 옵션
    docker-compose 는 기본적으로 'docker-compose.yml' 또는 'docker-compose.yaml' 의 이름을 사용, 만약 다른 이름으로 파일을 관리하고 사용한다면 다음과 같이 입력

    `docker-compose -f docker-compose-custom.yml up`
   - -d 옵션
    백그라운드에서 `docker-compose`를 실행하기 위해 사용

    `docker-compose up -d`

3. docker compose 용처
 docker compose는 주로 말아서 가동시킬때 사용하기 보다, db, redis 인프라 환경 구축시 로컬에다 하는 것보다. Image를 가져오는것보다 내리고 쓰거나 할떄 주로 사용함