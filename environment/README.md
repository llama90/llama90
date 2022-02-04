# Environment

## Linux Machine

### 사양
* CPU: [Intel Xeon Processor L5640](https://ark.intel.com/content/www/kr/ko/ark/products/47926/intel-xeon-processor-l5640-12m-cache-2-26-ghz-5-86-gts-intel-qpi.html) (x2)
* RAM: Hynix DDR3 PC3-8500 ECC 16G (x6)
* M/B: [Supermicro X8DAH+ DS&G](https://www.supermicro.com/products/motherboard/QPI/5500/X8DAH_.cfm)
* VGA: Integrated 
* SSD: 
  * [리눅스 디스크 I/O 성능 테스트하기. (Feat. dd / hdparm)](https://svrforum.com/os/112595)
    * Read/Write 모두 200 MB/s - 보드가 SATA2 까지 지원이라 SATA3 지원 카드 추가 필요
* HDD: 

### 설치 프로그램
* O/S: Ubuntu 20.04 LTS

#### O/S 설치 후 SSH 연결을 위한 패키지 설치

```shell
sudo apt-get install net-tools # ifconfig 명령어를 이용해 머신에 할당된 IP 확인을 위함
sudo apt-get install ssh # SSH 연결을 위함
```

#### Docker 설치
* [Ubuntu 20.04 LTS ) Docker 설치하기](https://shanepark.tistory.com/237)
* [Linux, sudo 없이 명령어 실행하기 (예:docker)](https://shanepark.tistory.com/250)
