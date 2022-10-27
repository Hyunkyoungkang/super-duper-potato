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
            
            ![그림2.jpg](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EA%25B7%25B8%25EB%25A6%25BC2.jpg)
            
        - 캐싱 전용 서버는 내부에 도메인이름이 없을때 외부로 나가 찾아서 연결 해주는 서버
        >일반적으로 쓰는 내부 DNS 서버
            
            ![그림3.jpg](DNS_server%20db9b5c585c564af1888f950e7ed5e7fb/%25EA%25B7%25B8%25EB%25A6%25BC3.jpg)
            
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