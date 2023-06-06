# Http 부하분산

```
upstream backend {                          # upstream 모듈
    server 127.0.0.1:80          weight1;   # server 지시자
    server app.example.com:      weight2;
    server spare.exampe.com:80   backup;
}
server {                                    # server모듈 
    location / {
        proxy_pass http://backend;
    }
}
```

80번 포트를 사용하는 HTTP 서버 두 대로 부하를 분산한다.
설정한 프라이머리(primary)<sup>[1](#footnote_1)</sup> 서버 두 대에 문제가 발생해 연결이 불가능 하면 backup으로 지정한 서버를 사용한다.

지정한 weight 매개변숫값에 따라 두 번째 서버는 첫 번째 서버보다 두 배 많은 요청을 받는다.
<br>
❗ : weight는 기본값이 1
<br>
❗ : weight는 우선순위가 아니라 가중치다.
<br>
<br>
## http **upstrem** module
Http 프로토콜 요청에 대한 부하분산 방식을 저의한다.
부하분산을 위한 목적지 pool<sup>[2](#footnote_2)</sup>은 유닉스 소켓, IP 주소, DNS레코드 혹은 이들의 조합으로 구성한다.
<br>
각 upstream 대상은 server 지시자로 설정한다. server 지시자는 앞서 언급한 유닉스 소켓, IP주소, 전체 주소 도메인 네임(FQDN) 형식으로 표시된 도메인 정보 등을 몇 가지  추가 매개변후와 함꼐 지정한다. 매개변수는 요청을 적절한 목적지로 전달하도록 추가 제어 방법을 제공한다.

이러한 매개변수는 분산알고리즘에 가중치를 적용하거나 서버가 **어떤 상태 / 가용 여부 인지** 알려준다.

✔tip nginx plus는 고급 매개변수를 제공, 특정 서버로 연결 수를 제한 및 고급DNS 질의를 제어하고 가동된지 얼마 안 된 서버로는 연결이 천천히 늘게 할 수 있다.



---
<a name="footnote_1">1</a> server지시자 두개에 weight 매개변수로 가중치를 설정한 
서버
<br>
<a name="footnote_2">2</a> 여러 대의 서버를 지칭하는 용어, 서버 그룹 혹은 서버 팜이라고도 한다.

<br>
<br>
<br>

# TCP 부하분산

### ⭕ code 예시1
```
stream{
    upstream mysql_read {                         
        server read1.exampe.com:3306       weight5;
        server read1.example.com:3306      
        server 127.0.0.1:3306              weight1;  
    }
    server {                                    
        listen 3306;
        proxy_pass mysql_read;
    } 
}

```
리드 레플리카(읽기전용 복제본) 두 대로 구성된 mysql 서버로 부하를 분산.

nginx 설치 전후 특별히 설정을 바꾸지 않았다면, 기본설정 파일 경로 conf.d 폴더는 http블록에 포함.

✅ stream 모듈을 이용한 이 설정은 stream.conf.d라는 별도의 폴더를 생성해 저장하는 편이 좋다.
경로 nginx.conf 파일의 strean불록에 추가해 nginx가 참조하도록 하자.
--> 예)
```
user nginx;
worker_preoceses auto;
pid /run/nfinx.pid;

stream{
    include /etc/nginx/stream.conf.d/*.conf;
}
```

/etc/nginx/stream.conf.d/mysql_read 설정파일
```
upstream mysql_read {                         
    server read1.exampe.com:3306       weight5;
    server read1.example.com:3306      
    server 127.0.0.1:3306              weight1;  
}
server {                                    
    listen 3306;
    proxy_pass mysql_read;
} 
```
즉,