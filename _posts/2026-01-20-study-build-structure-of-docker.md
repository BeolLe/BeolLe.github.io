---
layout: post
title: "도커 기본구조에 대해 공부 및 실습하기"
categories: enigneer
tags: [Cloud, Docker, knowledge, Engineer, build]
published: true
---

도커는 공간을 격리해서 찌꺼기를 남기지 않고 깔끔한 실행환경을 만들어 준다.또한 iptables라는 규칙을 통해 리눅스 내에서 방화벽을 뚫고 가장 최상위단에 도커 내부의 프로그램을 실제 컴퓨터 혹은 서버의 포트를 직접 노출해서 보안이 뚫려버릴 수 있다. 따라서 보안에 문제가 생길 수 있어 실제 서비스에서는 false값으로 두거나, 로컬호스트에 강제 바인딩(컴퓨터 내부에서 호출할 때만 작동하도록 하는 것)을 걸어버린다. 

이 보안을 위해서 클라우드 플레어같은 방어 서비스 > 클라우드 방화벽 > nginx > 방화벽 > 도커 네트워크(앱 > 디비)와 비슷한 순서로 이루어진다.

컨테이너는 찌꺼기를 남기지 않는데, 즉 컨테이너는 삭제하면 내부의 데이터가 전부 사라진다는 뜻이다. 

그래서 내부의 파일을 남기기 위해서는 별도의 장치가 필요한데 이를 마운트라고 한다.

마운트는 크게 두 개의 종류가 있다.

1. 바인드 마운트 - Host 경로를 직접 연결하는 방식으로, 사용자가 수시로 접근하여 수정해야 하는 파일(설정파일)을 연결하면 좋다.
2. 도커 볼륨 - 도커가 관리하는 저장소이며, 권한 문제나 안정성이 중요한 데이터(DB)를 연결하면 좋다.

또한, 물리디스크가 하나라면 도커를 통해 컨테이너를 여러 개 사용해도, 속도는 개선되지 않는다. 하나의 볼륨을 여러 DB가 공유하면 더욱 대기열이 생성되며 성능에 저하가 발생한다.

이를 해결하기 위해 샤딩이라는 방식이 있는데, 여러 개의 서버(물리적 디스크)에 나누어 저장하는 방식으로 대표적인 2가지 방식이 존재한다

1. Range Sharding - 특정 범위의 값으로 나누어 서버 별로 분산 저장하는 방식이다. 데이터 쏠림 현상이 발생하기 쉬움(Skew/Hotspot)
2. Hash Sharding - ID % 서버수 연산을 해서 균등분배한다. 트래픽을 분산하는데 유리하다.

하지만 이런 샤딩에도 단점은 존재한다. 샤딩키가 아닌 조건으로 검색하게 되면 모든 서버를 다 조회하면서 성능의 저하가 발생한다는 점이다. 

따라서 이를 해결하기 위해 CQRS(Command Query Responsibility Segregation)방식을 사용한다. 

즉 읽기, 쓰기를 나누어 진행하는데 이런 방식으로 진행된다

1. DB - 주로 쓰기를 담당하며 ID 기준으로 샤딩 진행. ACID 보장되어야 함. 단 ID 기준으로 읽을때는 DB가 빠르다.
2. Search Engine - 읽기를 주로 담당하며 ElasticSearch 등을 활용한다. ID 뿐 아니라 이름, 이메일 등을 역색인하여 빠르게 DB에서 ID를 검색할 수 있게 도와준다.

위의 공부를 바탕으로 아래와 같이 실습을 진행했다.

실습을 진행하는 주 목적은, 내가 블로그를 운영하면서 실시간으로 로그를 받아 볼 수 있는 환경을 도커로 구축하고, 이로 인해 발생되는 정보로 분석 혹은 파이프라인을 구축해서 dbt를 직접 사용해보는 것이다.

물론, 내 개인 블로그는 다른 사람들에게 대놓고 이야기한 적이 없기에 로그가 많이 쌓이지는 않을 것이다.

따라서, 스스로 Ddos에 가까운 환경을 만들어야하지 않을까...싶기는 하지만 우선은 하나하나 공부하기로 했다.

1. 디렉토리 및 환경 설계
2. yml 파일 작성
3. django를 사용해서 api처럼 작동하게 하기
4. db에 장고를 연결해서 실시간 블로그 로그 쌓이게 하기.

자 우선 난 여기서 헷갈리는 점을 먼저 Gemini와 짚고 넘어갔다.

### 1. 장고와 FastAPI, nginx는 어떤 차이가 있는가?
    - 장고 : 파이썬의 프레임워크 중 보안, DB, 관리자 페이지 등 모든 것이 적혀있지만 무겁고, 공부할 내용이 많고 검증된 보안 로직은 있지만, 그만큼 공부할 내용도 많음
    - FastAPI : 파이썬의 또다른 프레임워크 중 하나로, 속도가 매우 빠르며 비동기에 최적화 된 기술. 코드가 간결하지만 보안을 챙겨야 할 부분이 있음.
    - nginx : 외부에서 접속을 시도하면, 특정 포트로 연결시켜주는 기술.

    - 추가적으로 gunicorn : 파이썬 코드를 nginx가 이해할 수 있게 실행해주는 WSGI 서버

그래서 처음에 nginx를 app에 넣고 이미지로 이를 돌리려고 하던 제미나이를 막고, app에 파이썬 11 slim 버전을 넣고, 장고와 mysqlclient, gunicorn을 설치해서 db와 장고, 구니콘을 쓰고, depends_on 옵션을 써서 nginx가 장고가 켜지면 역할하도록 만들었다.

추가적으로 도메인 설정도 했고, 로컬에 블로그 돌릴 수 있게 또 만들었는데, 이는 git 관련이라 다음에 하도록 하겠다.

하지만! 이렇게 하다보니 문제가 생겼다. 

바로, 기존에 깔려 있는 것들을 토대로 사용하다보니, 지속적으로 pip install django-cors-headers같이 지속적으로 추가 라이브러리를 설치해야한다는 점이 불편했다.
그래서 DockerFile을 추가로 만들어서 이 안에 기본적인 설정을 넣고 다시 진행했다.

이 과정 중에 비록, 마운트까지 했지만 설정해둔 파일들이 날라가서 다시 설정했지만, 그래도 나름 즐거운 경험이었다.

그래서 결국 약 6시간의 씨름 끝에 db연결도 하고, 로그 적재되는 것 까지 확인을 했다.

코드 파일은 올리지 않겠지만, 기본적으로 docker-compose.yml의 초기 버전과, nginx/default.conf 파일은 여기에 올려두겠다.

gemini한테 부탁해서 각종 주석을 달아놓았기 떄문에, 추후에 다시 보면서 공부할 수 있기 때문이다.


```
# 도커 컴포즈 문법 버전 (3.8은 안정적인 표준 버전)
version: '3.8'

services:
  # [1. 문지기 서비스] 외부 요청을 가장 먼저 받는 Nginx
  nginx:
    image: nginx:latest           # 최신 Nginx 이미지 사용
    container_name: ronny-gate    # 컨테이너 이름을 'ronny-gate'로 고정
    ports:
      - "80:80"                   # 외부 80포트와 컨테이너 80포트를 연결 (웹 접속 허용)
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro # 설정파일 마운트 (ro: 읽기전용)
    networks:
      - ronny-server              # 동일한 가상 네트워크에 소속
    depends_on:
      - app                       # 장고(app)가 먼저 실행된 후 Nginx 실행

  # [2. 일꾼 서비스] 실제 로직을 처리하는 Django(Python)
  app:
    image: python:3.11-slim       # 파이썬 3.11 슬림 버전 이미지 사용 (가볍고 빠름)
    container_name: ronny-django  # 컨테이너 이름을 'ronny-django'로 고정
    working_dir: /app             # 컨테이너 내부 작업 디렉토리를 /app으로 설정
    volumes: 
      - ./app:/app                # 내 SSD의 소스코드 폴더와 컨테이너 내부 폴더 동기화
    command: >                    # 컨테이너 실행 시 수행할 명령어 뭉치
      sh -c "pip install -r requirements.txt && 
             if [ ! -f manage.py ]; then django-admin startproject myproject .; fi && 
             python manage.py migrate && 
             gunicorn myproject.wsgi:application --bind 0.0.0.0:8000"
      # 1) 패키지 설치 -> 2) 파일 없으면 장고 프로젝트 생성 -> 3) DB 구조 생성 -> 4) 서버 가동(8000번 포트)
    networks:
      - ronny-server              # 동일한 사설망에 소속
    depends_on:
      - db                        # 데이터베이스(db)가 먼저 준비되어야 함

  # [3. 금고 서비스] 데이터를 영구 저장하는 MySQL
  db:
    image: mysql:8.0              # MySQL 8.0 이미지 사용
    container_name: ronny-server-db
    environment:
      MYSQL_ROOT_PASSWORD: 너는바보다 # DB 관리자 비밀번호 (실습용)
      MYSQL_DATABASE: ronny_db            # 자동으로 생성할 DB 이름
    volumes:
      - ./db_data:/var/lib/mysql  # [핵심] DB 데이터를 내 SSD(db_data)에 저장하여 영구 보존
    networks:
      - ronny-server              # 사설망 소속 (외부 직접 접속 차단됨)

# 네트워크 설정: 서비스들끼리 이름으로 통신할 수 있게 해주는 가상 랜선
networks:  
  ronny-server:
    driver: bridge                # 이 호스트 내부에서만 통하는 브릿지 드라이버 사용
```

default.conf 파일 내용
```
# Nginx 서버 설정 블록
server {
    listen 80;                # 80번 포트(기본 웹 접속)에서 대기
    server_name localhost;    # 접속 주소가 localhost인 경우 아래 규칙 적용

    # 모든 요청('/')에 대한 처리 규칙
    location / {
        # [리버스 프록시] 요청을 내부망에 있는 'app' 서비스의 8000번 포트로 전달
        proxy_pass http://app:8000; 
        
        # 보안 및 분석을 위해 원본 요청 헤더 정보를 전달
        proxy_set_header Host $host;                          # 원래 호스트 정보 유지
        proxy_set_header X-Real-IP $remote_addr;             # 접속자의 실제 IP 전달
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # 경유지 정보 누적
    }
}
```

추후에 더욱 공부할 내용
1. 장고의 기본 작동 원리
2. FastAPI의 기본적인 원리 및 작동 방식
3. gunicorn의 정확한 역할 및 작동 원리