title: Mac terminal 명령어를 통한 시스템 모니터링
description: Mac terminal 명령어를 통한 시스템 모니터링
categories:
 - labs
tags: system
---

### 와탭 기술 블로그 [리눅스 명령어를 통한 시스템 모니터링](http://tech.whatap.io/2015/09/03/linux-monitoring/#comment-914)을 참고하여 Mac 터미널 명령어와 일부 다른 내용을 정리했습니다.  

### Mac terminal 명령어를 통한 시스템 모니터링 
#### $ uname -a
- / wiki / [uname](https://ko.wikipedia.org/wiki/Uname) : 시스템과 커널의 정보 확

시스템 또는 커널 (-s) | 호스트명 | 운영 체제 또는 배포판 (-o) | 릴리즈정보(-r) | 커널버전(-v) | 머신하드웨어이름 (-m) | 하드웨어플랫폼(-i or -M) | 
|---|---|---|---|---|---|---|
Darwin | i-sang-u.local | 유효하지 않은 명령 | 17.6.0 | Darwin Kernel Version 17.6.0: Tue May 8 15:22:16 PDT 2018; root:xnu-4570.61.1~1/RELEASE_X86_64 | x86_64 | 유효하지 않은 명령

#### $ ifconfig
- / doc.oracle.com / [ipconfig](https://www.ibm.com/support/knowledgecenter/ko/ssw_aix_71/com.ibm.aix.cmds3/ifconfig.htm) :시스템에 설정된 네트워크 인터페이스의 상태를 확인, 변경

- `ifconfig -a` : 시스템에 있는 모든 인터페이스의 모든 주소를 봅니다.
- `ifconfig -a4` : 시스템의 인터페이스에 지정된 모든 IPv4 주소를 봅니다.
- `ifconfig -a6` : 로컬 시스템에서 IPv6이 사용으로 설정된 경우 시스템의 인터페이스에 지정된 모든 IPv6 주소를 표시합니다.
- (curl 설치 후) `curl ifconfig.me` : 공인 ip를 포함한 인터페이스 목록.
-  `ifconfig` : 현재 사용중인 인터페이스 목록
    ````
    ...
    
    lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
        options=1203<RXCSUM,TXCSUM,TXSTATUS,SW_TIMESTAMP>
        inet 127.0.0.1 netmask 0xff000000 
        inet6 ::1 prefixlen 128 
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 
        nd6 options=201<PERFORMNUD,DAD>
    
    ...(more)
    
    ````
    변수 | 화면 출력 | 설명 
    |---|---|---|
    인터페이스 이름 | lo0 | ifconfig 명령에서 상태가 요청된 인터페이스의 장치 이름을 나타냅니다.
    인터페이스 상태 | flags=8049<UP | 현재 인터페이스와 연결된 플래그를 포함하여 인터페이스의 상태를 표시합니다. 인터페이스가 현재 초기화되었는지(UP) 또는 초기화되지 않았는지(DOWN ) 여부를 확인할 수 있습니다.
    브로드캐스트 상태 | LOOPBACK | 인터페이스에서 IPv4 브로드캐스트를 지원한다는 것을 나타냅니다.
    전송 상태 | RUNNING | 시스템에서 인터페이스를 통해 패킷을 전송한다는 것을 나타냅니다.
    멀티캐스트 상태 | MULTICAST, (IPv4, BROADCAST...) | 인터페이스에서 멀티캐스트 전송을 지원한다는 것을 표시합니다. 
    최대 전송 단위 | mtu 16384 | 이 인터페이스의 최대 전송 크기가 16384옥테트임을 표시합니다.
    IP 주소 | inet 127.0.0.1 | 인터페이스에 지정된 IPv4 또는 IPv6 주소를 표시합니다. 
    넷마스크 | netmask 0xff000000 | 특정 인터페이스의 IPv4 넷마스크를 표시합니다. IPv6 주소는 넷마스크를 사용하지 않습니다.
    (MAC 주소) | (ether xx:xx:xx:xx:xx:xx) | 인터페이스의 이더넷 계층 주소를 표시합니다.
    
#### $top
- [활성상태보기](https://support.apple.com/ko-kr/ht201464)는 top 커맨드의 gui.
- / blogger / [top](https://docstore.mik.ua/orelly/unix3/mac/ch08_01.htm) : 프로세스 작업 명령어. 실시간 시스템 프로세스들의 CPU/Memory 점유율.

#### $vm_stat
- 시스템 작업, 하드웨어 및 시스템 정보를 확인. 메모리, 페이징, 블록장치의 I/O, CPU상태 정보.
- Linux `free` 명령어를 mac 명령어로 바꾸기 : [apple.stackexchange](https://apple.stackexchange.com/questions/4286/is-there-a-mac-os-x-terminal-version-of-the-free-command-in-linux-systems)
- `vm_stat` 명령어를 파싱해서 free 명령어와 유사한 결과를 출력합니다.
    ```` 
    $vm_stat | perl -ne '/page size of (\d+)/ and $size=$1; /Pages\s+([^:]+)[^\d]+(\d+)/ and printf("%-16s % 16.2f Mi\n", "$1:", $2 * $size / 1048576);'
    
    free:                     3066.55 Mi
    active:                   7034.39 Mi
    inactive:                 1979.59 Mi
    speculative:               928.85 Mi
    throttled:                   0.00 Mi
    wired down:               2568.95 Mi
    purgeable:                 430.81 Mi
    copy-on-write:          155584.08 Mi
    zero filled:           2841792.27 Mi
    reactivated:             15422.36 Mi
    purged:                  25047.20 Mi
    stored in compressor:          3227.00 Mi
    occupied by compressor:           803.12 Mi
    ````
    
- `$ vm_stat 15#` : 1초 인터벌을 갖고 모니터링 정보 5개를 출력하고 커맨드 종료

    free | active  | specul |inactive |throttle    |wired  |prgable   |faults     |copy    |0fill |reactive   |purged |file-backed |anonymous |cmprssed |cmprssor  |dcomprs   |comprs  |pageins  |pageout  |swapins |swapouts
    ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
    765099|  1824829   |236456   |507908        |0   |653905   |109441 |1144088K |39924576  |729154K  |3948809  |6412090      |938997   |1630196   |825991   |205645 |26519556 |38398651 |17467981    |11486 |25642145 |26240891|
    ...(1초 인터벌, 5번 출력)

#### $iostat 
- / docs.oracle / [iostat](https://docs.oracle.com/cd/E24846_01/html/E23088/spmonitor-4.html) : iostat 명령을 사용하여 디스크 입출력에 대한 통계를 보고하고 처리량, 사용률, 대기열 길이, 트랜잭션 비율 및 서비스 시간에 대한 측정 결과를 표시
    ````
    $ iostat 5
         tty          fd0           sd3          nfs1         nfs31          cpu
    tin tout kps tps serv  kps tps serv  kps tps serv  kps tps serv  us sy wt id
      0    1   0   0  410    3   0   29    0   0    9    3   0   47   4  2  0
    ... (5초 인터벌)
    ````
    - 출력의 첫 라인은 시스템이 마지막으로 부팅된 이후의 통계. 이후 각 라인은 간격 통계
