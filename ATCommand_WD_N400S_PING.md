# Cat.M1 외장형 모뎀 WD-N400S의 PING 테스트 가이드

## 목차

-   [소개](#Step-1-Overview)
-   [AT 명령어](#Step-2-ATCommand)
-   [PING 테스트](#Step-3-Test)

<a name="Prerequisites"></a>


### Development Environment
* **시리얼 터미널 프로그램** ([Token2Shell](https://choung.net/token2shell), [PuTTY](https://www.putty.org), [TeraTerm](https://ttssh2.osdn.jp) 등)

- **TCP Server 프로그램** ([Hercules](https://www.hw-group.com/software/hercules-setup-utility) 등)



### Hardware Requirement

* [**외장형 Cat.M1 모뎀(WD N400S)**](http://wiznetshop.co.kr/product/detail.html?product_no=786)

<img src = "./imgs/wd-n400s.PNG" width="400px">

* [**외장형Cat.M1(WD N400S) Interface B/D **](http://wiznetshop.co.kr/product/detail.html?product_no=787)

  <img src = "./imgs/wd-n400s_if.PNG" width="300px">


* [**외장형 Cat.M1 확장 Cable **](http://wiznetshop.co.kr/product/detail.html?product_no=928)

  <img src = "./imgs/wd-n400s_cable.PNG" width="300px">
  
* [**Micro USB Cable **](http://wiznetshop.co.kr/product/detail.html?product_no=791)

  <img src = "./imgs/micro_cable.JPG" width="200px">

<a name="Step-1-Overview"></a>

## 소개
본 문서에서는 Cat.M1 단말인 우리넷 외장형 모뎀의 Ping 테스트 방법에 대한 가이드를 제공합니다.

외장형 모뎀은 UART 인터페이스를 통해 활용하는 AT 명령어로 제어하는 것이 일반적입니다. Cat.M1 모듈 제조사에 따라 AT 명령어의 차이는 있지만,  일반적인 Ping 테스트를 위한 통신 과정은 다음과 같은 순서로 구현합니다. 

1. Echo 모드 설정
2. USIM 상태 확인
3. 네트워크 접속 확인
4. 네트워크 인터페이스(PDP Context) 활성화
5. Ping 테스트

추가적으로, TCP 가이드 문서에는 다른 응용 가이드 문서에는 포함되어 있지 않은 Cat.M1 단말의 상태 확인 및 PDP context 관련 명령어에 대한 내용이 함께 포함되어 있습니다. 해당 명령어는 응용 구현 시 필수적으로 활용되어야 하므로, 함께 확인하시기 바랍니다.
* Echo 모드 설정: `ATE`
* USIM 상태 확인: `AT$$STAT?`
* 망 등록 및 상태 점검: `AT+CEREG`
* PDP Context 활성화 및 비활성화: `AT*RNDISDATA`
<a name="Step-2-ATCommand"></a>
## AT 명령어

### 1. Echo 모드 설정

ATE0로 설정되면 입력된 명령어 Echo back이 비활성화 됩니다.
MCU board로 Cat.M1 모듈을 제어하는 경우 해당 명령어를 사용합니다.

**AT Command:** ATE

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | ATE(value1) | OK | ATE0<br>OK |

**Defined values:**

| Parameter | Type    | Description                           |
| :-------- | :------ | :------------------------------------ |
| (value1)  | integer | 0 : Echo mode OFF<br>1 : Echo mode ON |

### 2. USIM 상태 확인
이 명령어는 USIM의 Password를 입력하거나 password 입력이 필요 없는 경우 USIM의 정상 운용이 가능한 상황인지 확인합니다. 본 가이드에서는 password가 없는 상황에서 USIM 상태를 확인하기 위해 사용합니다.
> **READY** 응답이 출력되면 정상입니다.

**AT Command:** AT$$STAT?

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Read | AT$$STAT? | $$STAT:(value1) | AT$$STAT?<br>$$STAT:READY<br><br>OK |

**Defined values:**

| Parameter | Type   | Description                                                  |
| :-------- | :----- | :----------------------------------------------------------- |
| (value1)  | string | INSERT : USIM card inserted<br>OPEN : MSISDN NULL<br>READY : 정상적으로 Card Initialization 마친 상태<br>MNCCARD : 다른 사업자 USIM<br>MCCRAD : 해외 사업자 USIM<br>TESTCARD : 장비 테스트 USIM<br>ONCHIP : onChip SIM mode<br>PIN : SIM PIN, SIM PUK 등 남은 시도 횟수<br>NET PIN : PLMN ID 이외의 값을 가진 카드<br>SIM PERM BLOCK : PUK 모두 실패. 카드 교체 필요<br>SIM PIN VERIFIED : PIN code 입력 성공<br>SIM PERSO OK : Personalization unlock 성공<br>FAILURE, REMOVED : USIM removed<br>FAILURE,NO_CARD : USIM 삽입 안됨<br>AILURE,ERROR : USIM인식 Error |

### 3. 망 등록 및 상태 점검

망 서비스 상태 확인을 위해 사용되는 명령어 입니다. 디바이스 구현 시, 망 연결 유지를 위해 주기적으로 체크하는 것을 권장합니다.

**AT Command:** AT+CEREG?

**Syntax:**


| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Read | AT+CEREG? | +CEREG: (value1),(value2)<br><br>OK | AT+CEREG?<br>+CEREG: 0,1<br><br>OK |

**Defined values:**

| Parameter |  Type   | Description                                                  |
| :-------- | :-----: | :----------------------------------------------------------- |
| (value1)  | integer | 0 : Disable network registration unsolicited result code<br>1 : Enable network registration unsolicited result code<br>2 : Enable network registration and location information unsolicited result code<br>4 : For a UE that wants to apply PSM, enable network registration and location information unsolicited result code |
| (value2)  | integer | 0 : Not registered. MT is not currently searching an operator to register to.<br>1 : Registered, home network<br>2 : Not registered, but MT is currently trying to attach or searching an operator to register to.<br>3 : Registration denied<br>4 : Unknown<br>5 : Registered, roaming |

### 4. PDP Context 활성화 및 비활성화
> PDP(Packet Data Protocol)란 단말과 외부 패킷 데이터 네트워크 사이의 데이터 송수신을 위한 연결을 제공하기 위해 사용하는 네트워크 프로토콜을 뜻하며, PDP Context는 이러한 연결 과정에서 사용되는 정보의 집합을 의미합니다.

**AT Command:** AT*RNDISDATA

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | AT*RNDISDATA=(value1) | *RNDISDATA=(value2)<br><br>OK | AT\*RNDISDATA=1<br>*RNDISDATA:1<br><br>OK |

**Defined values:**

| Parameter | Type    | Description                                                  |
| :-------- | :------ | :----------------------------------------------------------- |
| (value1)  | integer | 0 : RNDIS Device 미사용(전화 접속 연결 사용)<br>1 : RNDIS Device 사용(전화 접속 연결 사용 불가) |

| Parameter | Type    | Description                                                  |
| :-------- | :------ | :----------------------------------------------------------- |
| (value2)  | integer | 0 : RNDIS Device 미사용(전화 접속 연결 사용)<br>1 : RNDIS Device 사용(전화 접속 연결 사용 불가) |


### 5. Ping 테스트

**AT Command:** AT*PING

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | AT*PING=(value1),(value2) | OK | AT*PING=8.8.8.8,3"<br><br>OK<br>PING 64:ff9b::8.8.8.8(64:ff9b::808:808) 56 data bytes<br>64 bytes from 64:ff9b::808:808: icmp_seq=1 ttl=109 time=148 ms<br>64 bytes from 64:ff9b::808:808: icmp_seq=2 ttl=109 time=104 ms<br>64 bytes from 64:ff9b::808:808: icmp_seq=3 ttl=109 time=142 ms<br><br><br>--- 64:ff9b::8.8.8.8 ping statistics ---<br>3 packets transmitted, 3 received, 0% packet loss, time 2009ms<br>rtt min/avg/max/mdev = 104.895/131.917/148.183/19.239 ms<br>rtt min/avg/max/mdev = 104.895/131.917/148.183/19.239 ms |


**Defined values:**

| Parameter | Type | Description |
|:--------|:--------|:--------|
| [value1] | String | Host address (URL, IPv4, IPv6) |
| [value2] | Integer | Ping 송신 횟수 |



<a name="Step-3-Test"></a>

## Ping Test 

### 1. 하드웨어 연결

- 모뎀을 PC와 Serial로 연결한 후 COM Port Number를 확인합니다.

![](./imgs/wd-n400s_connect.JPG)

![1619684840712](./imgs/Example_01.PNG)



### 2. 모듈 상태 확인

- 확인한 COM Port Nubmer로 Serial Terminal 프로그램을 실행하여 Open합니다. TCP Socket을 생성하기전에 아래의 명령어를 수행합니다.


```
// Serial 통신 확인
AT
OK

// AT 명령어 echo 비활성화
ATE0
OK

// USIM 상태 확인 (READY : 정상)
AT$$STAT?
$$STAT:READY

OK

// 망 접속 확인
AT+CEREG?
+CEREG: 0,1

OK

// PDP context 활성화
AT*RNDISDATA=1
*RNDISDATA:1

OK
```
![](./imgs/Example_02.PNG)



### 3. PING 테스트

- PING TEST 를 진행할 IP와 횟수를 설정하고 결과를 확인합니다.
```
//Ping Test
AT*PING=8.8.8.8,3<br>
<br>
OK<br>
PING 64:ff9b::8.8.8.8(64:ff9b::808:808) 56 data bytes<br>
64 bytes from 64:ff9b::808:808: icmp_seq=1 ttl=109 time=148 ms<br>
64 bytes from 64:ff9b::808:808: icmp_seq=2 ttl=109 time=104 ms<br>
64 bytes from 64:ff9b::808:808: icmp_seq=3 ttl=109 time=142 ms<br>
<br>
--- 64:ff9b::8.8.8.8 ping statistics ---<br>
3 packets transmitted, 3 received, 0% packet loss, time 2009ms<br>
rtt min/avg/max/mdev = 104.895/131.917/148.183/19.239 ms<br>
rtt min/avg/max/mdev = 104.895/131.917/148.183/19.239 ms<br>
```
![](./imgs/PING_Example_01.PNG)

