# DNS_server

생성일: 2022년 10월 26일 오후 7:39

## DNS_server

- DNS_server 서버란?
    - 도메인 이름을 IP주소로 변환시켜주는 서버
    WWW.NAVER.COM > 192.168.0.0…..
    - 초창기에는 서버 컴퓨터가 몇개없었기떄문에 단순했지만 지금은 네트워크가 생겨나면서 셀수없을만큼의 서버들이 있기때문에 일일히 숫자로 외우기엔 한계가있다. 그래서 숫자보단 단어로 외우기가 쉽기때문에 IP주소를 도메인이름으로 바꾸고 그걸 다시 IP주소로 바꿔준다.
    - 쉽게 생각하면 전화번호부 같은 기능이라고 보면된다.
    - 기본적으로 IP주소를 찾을때 내부 전화번호부를 먼저 찾는다. /etc/hosts
    
    ![hosts파일 수정.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/hosts%25ED%258C%258C%25EC%259D%25BC_%25EC%2588%2598%25EC%25A0%2595.jpg)
    
    ```html
    hosts파일은 내가 전화번호부를 직접 작성
    여기에 없으면 resolv.conf파일을 찾는다.
    ```
    
    - 내부네임서버 설정파일이 없다면 여기로 와서 찾게된다.
        
        ![네임서버 설정파일.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EB%2584%25A4%25EC%259E%2584%25EC%2584%259C%25EB%25B2%2584_%25EC%2584%25A4%25EC%25A0%2595%25ED%258C%258C%25EC%259D%25BC.jpg)
        
        ```html
        resolv.conf파일은 다른 전화번호부(네임서버)를 연결
        ```
        
    - IP주소 찾는 과정
        
        ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1.jpg)
        
    - 로컬네임서버 동작과정
        
        ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%201.jpg)
        
- DNS_server 서버 만들기
    - 기본적으로 네임서버는 두가지로 나뉜다
        - 마스터 네임 서버는 실제 서비스 구축 후 외부에 알려줄떄 사용하는 서버
        >자신이 구축한 서버에 접속하기 위한 네임서버
            
            ![그림2.png](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EA%25B7%25B8%25EB%25A6%25BC2.png)
            
        
        - 캐싱 전용 서버는 내부에 도메인이름이 없을때 외부로 나가 찾아서 연결 해주는 서버
        >일반적으로 쓰는 내부 DNS 서버
            
            ![그림3.png](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EA%25B7%25B8%25EB%25A6%25BC3.png)
            
    - 캐싱 전용 서버
        1. 네임서버 설치 bind라는 데몬 프로그램이 대표적이다.
            
            ![캐싱전용네임서버 설치.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EC%25BA%2590%25EC%258B%25B1%25EC%25A0%2584%25EC%259A%25A9%25EB%2584%25A4%25EC%259E%2584%25EC%2584%259C%25EB%25B2%2584_%25EC%2584%25A4%25EC%25B9%2598.jpg)
            
            ```
            yum -y install bind-chroot
            ```
            
        2. /etc/named.conf 파일 설정(네임서버 설정정보)
        현재 테스트에는 총 4가지만 바꿈
        
        ```html
        listen-on port 53 { any; };
        listen-on-v6 port 53 { none; };
        
        allow-query { any; };
        
        dnssec-validation no;
        ```
        
        - 아래는 설정파일의 기능들을 정리해두었다.
        
        ```html
               # 네임서버에 접속이 허용된 IP 주소	
                listen-on port 53 { any; };
        				>이서버를 불특정 다수에게 서비스한다 >any
                >사설 DNS서버로 사용할땐 사설 IP 대역대를 고정
        
                listen-on-v6 port 53 { none; };
                > 위와같은 내용으로 IPv6 사용할떄 사용 (사용안하니 none)
        
               # 네임서버 DB 파일이 들어있는 디렉터리 
                directory       "/var/named";
                > 실제로 서비스할 DNS의 zone파일 디렉토리
        				> 실제 IP주소가 적여있는 파일이라고 생각하면됨
        
               # 정보가 갱신될 때 저장되는 파일 지정
                dump-file       "/var/named/data/cache_dump.db";
        				>일반적으로 운영할떄는 자주 사용안함
        
               # 통계 파일이 생성되는 파일 지정
                statistics-file "/var/named/data/named_stats.txt";
        				>일반적으로 운영할떄는 자주 사용안함
        
               # 메모리 관련 통계 파일이 생성되는 파일 지정
                memstatistics-file "/var/named/data/named_mem_stats.txt";
        				
                secroots-file   "/var/named/data/named.secroots";
                recursing-file  "/var/named/data/named.recursing";
                >일반적으로 운영할떄는 자주 사용안함
        
               # DNS 쿼리를 허용할 IP 설정
                allow-query     { any; };
        
               # 외부에서 네임서버 지정하여 허용/차단 옵션
                recursion yes;
        				>외부 접근여부 설정 보통 YES다.(사설망에 경우 NO를 표기)
        
               # dnssec 기능 활성화와 검증 설정
                dnssec-enable yes;
                dnssec-validation yes;
        
                managed-keys-directory "/var/named/dynamic";
        
                pid-file "/run/named/named.pid";
                session-keyfile "/run/named/session.key";
        
                include "/etc/crypto-policies/back-ends/bind.config";
        };
        
        logging {
        
               # 디버깅 로그 설정
                channel default_debug {
                        print-time yes;
                        print-category yes;
                        print-severity yes;
                        file "data/named.run";
                        severity dynamic;
                };
        
               # DNS 쿼리 로그 설정
                channel queries_log {
                        print-time yes;
                        print-category yes;
                        print-severity yes;
                        file "data/queries" versions 10 size 20M;
                        severity info;
                };
        
               # DNS 쿼리 에러 로그 설정
                channel query-errors_log {
                        print-time yes;
                        print-category yes;
                        print-severity yes;
                        file "data/query-errors" versions 10 size 20M;
                        severity dynamic;
                };
        
                category queries { queries_log; };
                category query-errors { query-errors_log; };
        };
        
        # 루트 도메인 지정
        zone "." IN {
        
               # 네임서버 타입 지정 : hint(루트도메인), master(1차 네임서버), slave(2차 네임서버)
                type hint;
                file "named.ca";
        };
        
        # 외부 설정파일 경로 지정
        include "/etc/named.rfc1912.zones";
        
        # root 영역을 위한 DNS Key 파일 경로 지정
        include "/etc/named.root.key";
        
        # 도메인 존 이름 지정
        zone "daum.net" IN {
        		
               # 1차 네임서버 지정
                type master;
        
               # 도메인 존 파일 지정
                file "daum.net.db";
                
               # 2차 네임서버 주소 지정, none은 2차 네임서버 없음
                allow-update { none; };
        ```
        
        1. 네임서버 시작 후 상태 확인
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%202.jpg)
            
        2. 방화벽 열기
            
            ![gui환경체크.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/gui%25ED%2599%2598%25EA%25B2%25BD%25EC%25B2%25B4%25ED%2581%25AC.jpg)
            
            ```html
            GUI환경에서 방화벽 열기 
            터미널에서 firewall-config 입력 시 설정창 뜸
            영역 > public > 서비스 > dns 클릭 후 위쪽 보기란에서 방화벽 다시열기 
            다시열었을때 체크가 되어있으면 방화벽이 열려있는상태
            ```
            
            ![ccl환경.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/ccl%25ED%2599%2598%25EA%25B2%25BD.jpg)
            
            ```html
            CCL환경에서 방화벽 열기 
            firewall-cmd --add --permanent --add-service=dns
            방화벽 열어줄 서비스 선택
            
            firewall-cmd --reload
            방화벽 설정내용 저장
            
            systemctl restart firewall
            방화벽 재시작 (굳이 안해도됨)
            ```
            
        3. 테스트
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%203.jpg)
            
            ```html
            nslookup 명령어로 서버 IP 변경 후 테스트 진행 
            정상적으로 동작하는걸 볼수있다.
            ```
            
        4. 네임서버를 영구적으로 적용시키기 위해 resolv.conf 파일 수정
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%204.jpg)
            
            ```html
            리눅스에서 명령어로 적용시키는 내용은 대부분 임시적인 부분이 많음으로 
            반드시 설정파일에 직접 수정한다
            ```
            
    - 라운드 로빈 서버
        - 하나의 서버를 사용하는 경우 사용자가 많을경우에 과부하가 걸릴수있다.
        - 이를 대비하여 똑같은 서버를 여러개 구축해놓고 번갈아가며 처리해준다.
        - 동시에 3명이 요청할 경우 1번 ,2번, 3번서버가 각각 한번씩 처리(분산처리)
        - 실제 테스트를 위해선 똑같이 셋팅된 서버가 여러개 필요하지만 여기 테스트에서는 서버한대로 여러개의 서버IP를 등록하여 테스트한다.
        1. nslookup으로 이미 구축되어있는 웹서버의 IP주소찾기
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%205.jpg)
            
            ```
            아무거나 아는 사이트주소 IP를 찾는다.최소 2개 이상.
            ```
            
        2. 마스터네임 서버 설정파일 수정
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%206.jpg)
            
        
        ```html
        기존 WWW 주소를 삭제 후 
        아래의 내용을 입력한다.
        www     IN	CNAME   webserver.john.com.
        >www.요청이 오면 webserver.john.com로 접속해라.
        >단, john.com는 접속할수있는 서버가 여러개있다.
        
        >www.john.com의 여러개 서버주소입력(실제는 같은 설정을 가진 서버들을 입력하면된다)
        webserver 1     IN	A	211.249.220.24
                  2     IN	A	120.50.131.112
                  3     IN	A	142.250.76.132
        ```
        
        1. 네임서버 재시작
        
        ```html
        systemctl restart named
        ```
        
        1. 설정된 내용 확인
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%207.jpg)
            
            ```html
            nslookup 명령어로 www.john.com 실행하면 3개의 IP가 뜬다.
            >도메인네임이 같은 3개의 DNS서버가 있다는 말
            ```
            
    - 마스터 네임 서버
        1. 테스트 진행을 위해 마스터네임서버에 웹서버도 같이 구축진행 
            
            ![웹서버 설치.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EC%259B%25B9%25EC%2584%259C%25EB%25B2%2584_%25EC%2584%25A4%25EC%25B9%2598.jpg)
            
            ```
            yum -y install httpd(아파치 웹서버 설치)
            ```
            
        2. 웹서버 방화벽 포트 열기
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%208.jpg)
            
        
        ```html
        firewall-cmd --permanent --add-service=http
        firewall-cmd --permanent --add-service=https
        -http 관련 서비스가 2가지가있음(보안관련내용인데 https가 더 보안이높음)
        GUI환경에서는 방화벽 설정 창 들어가서 설정화면됨
        
        터미널 명령시 firewall-config > 방화벽 설정창 뜸 >http,https 체크 후 끄기
        ```
        
        1. http 서버 재시작
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%209.jpg)
            
        
        ```html
        systemctl restart httpd
        
        systemctl enable httpd
        ```
        
        1. 테스트 페이지 만들기 
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2010.jpg)
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2011.jpg)
            
            ```html
            /var/www/html 안에 html 파일을 생성
            태그를 이용하여 테스트 페이지를 하나 만든다.
            웹서버접속 설정은 완료.
            ```
            
        2. 마스터네임 서버 설정
            
            ![마스터네임서버 설정경로.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EB%25A7%2588%25EC%258A%25A4%25ED%2584%25B0%25EB%2584%25A4%25EC%259E%2584%25EC%2584%259C%25EB%25B2%2584_%25EC%2584%25A4%25EC%25A0%2595%25EA%25B2%25BD%25EB%25A1%259C.jpg)
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2012.jpg)
            
            ```html
            /etc/named.conf 파일안에 내용을 변경한다.
            파일 내용중 제일 밑으로 가면 zone 설정이 나온다.
            > 여기서 zone이란 전화번호부를 가르키는 설정이라고 보면된다.
            
            zone "john.com" IN { 
                 >요청한 도메인의 이름
                    type master;
                    file "john.com.db";
                 >실제 IP주소가 적혀있는 정보파일
                    allow-update { none; };
            };
            
            ```
            
        
        1. 마스터네임서버 IP주소 목록 작성
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2013.jpg)
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2014.jpg)
            
            ```html
            도메인이름의 실제 IP주소를 쓰는 파일생성.
            /var/named 안에 .db파일을 만들면된다.
            (이름은 설정파일에 설정해놓은것과 똑같이 만들어야된다.)
            
            파일내용
            $TTL    3H
            @	SOA     @	[DNS서버주소.] ([zone파일계정번호] [보조DNS사용시 Refresh시간] 
                                        [보조DNS사용시 Retry시간] 
            														[보조DNS사용시, zone 파일 파기 유예기간] 
                                        [네거티브 캐시 유효기간] )
                    IN	NS	@
                    IN	A	[마스터네임 서버 주소]
            (john.com과 연관된 서버들 주소)
            www     IN	A	[웹서버 주소]
            ftp     IN	A	[파일서버주소]
            .....
            ```
            
        2. 설정 파일 이상유무 확인후 서비스 재시작
            
            ![설정파일 이상유무 확인.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EC%2584%25A4%25EC%25A0%2595%25ED%258C%258C%25EC%259D%25BC_%25EC%259D%25B4%25EC%2583%2581%25EC%259C%25A0%25EB%25AC%25B4_%25ED%2599%2595%25EC%259D%25B8.jpg)
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2015.jpg)
            
            ```html
            named-checkconf (마스터네임 설정파일 이상유무 체크)
            >이상없으면 아무것도 안뜸
            
            named-checkzone john.com john.com.db(db 파일 이상유무 체크)
            ```
            
        3. 테스트 하기
            
            ![1.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/1%2016.jpg)
            
            ![클라이언트 접속 확인.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25ED%2581%25B4%25EB%259D%25BC%25EC%259D%25B4%25EC%2596%25B8%25ED%258A%25B8_%25EC%25A0%2591%25EC%2586%258D_%25ED%2599%2595%25EC%259D%25B8.jpg)
            
            ![ftp서버 접속확인.JPG](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/ftp%25EC%2584%259C%25EB%25B2%2584_%25EC%25A0%2591%25EC%2586%258D%25ED%2599%2595%25EC%259D%25B8.jpg)
            
            ```html
            클라이언트 네트워크 설정에서 DNS서버를 변경
            변경 후 웹브라우저 켠 후 도메인이름으로 접속확인 www.john.com(웹서버)
            
            추가로 파일서버도 db등록했기떄문에 파일서버도 접속가능하다.
            ```