# Arduino 기반의 Cat.M1 HTTP 데이터 통신 가이드

## 목차

-   [시작하기 전에](#Prerequisites)
-   [소개](#Step-1-Overview)
-   [AT 명령어](#Step-2-ATCommand)
-   [동작 구조 예제](#Step-3-SampleCode)
-   [예제 코드 빌드 및 실행](#Step-4-Build-and-Run)

<a name="Prerequisites"></a>
## 시작하기 전에

> * 하드웨어 설정과 개발환경 구축은 **[Arduino 기반으로 Cat.M1 디바이스 개발 시작하기][arduino-getting-started]** 문서에 상세히 설명되어 있습니다.

> * Cat.M1과 같은 Cellular IoT 디바이스는 통신 서비스 사업자의 운영 기준 및 규정에 따라 모듈 펌웨어 및 동작 방식에 차이가 있을 수 있습니다. 본 문서는 한국 **[SK Telecom Cat.M1 서비스][skt-iot-portal]** 를 기준으로 작성되었습니다.

> * 아래 AT Command에 대한 설명은 HTTP 연동에 꼭 필요한 명령어만 설명하고 있습니다. 보다 자세한 설명은 Cat M1 모듈 매뉴얼을 참고하시기 바랍니다.

### Development Environment
* **[Arduino IDE][link-arduino-compiler]**

### Hardware Requirement

| MCU Board | IoT Shield Interface Board |
|:--------:|:--------:|
| [Arduino Mega2560 Rev3][link-arduino Mega2560 Rev3] | WIoT-WM01 (WM-N400MSE) |

<a name="Step-1-Overview"></a>
## 소개
본 문서에서는 Arduino IDE 기반 개발 환경에서 WIZnet IoT shield와 Arduino Mega2560 Rev3 보드를 이용하여 Cat.M1 단말의 TCP 데이터 송수신 방법에 대한 가이드를 제공합니다.

Cat.M1 모듈 및 외장형 모뎀은 UART 인터페이스를 통해 활용하는 AT 명령어로 제어하는 것이 일반적입니다. Cat.M1 모듈 제조사에 따라 AT 명령어의 차이는 있지만, 일반적인 TCP clint(UDP 포함)의 통신 과정은 다음과 같은 순서로 구현합니다.

1. 네트워크 인터페이스 활성화
2. HTTP 설정 - 목적지 URL, 옵션 
3. HTTP Request 전송
4. HTTP Response 확인
5. 네트워크 인터페이스 비활성화

우리넷 Cat.M1 모듈의 경우에는 현재 HTTP 관련 AT 명령어를 지원하지 않아 TCP 관련 AT 명령어를 이용하여 HTTP 메소드를 사용하는 방법으로 작성되었습니다.

<a name="Step-2-ATCommand"></a>
## AT 명령어

> 좀 더 상세한 AT 명령어 설명은 우리넷의 AT Command Manual에서 확인 하실 수 있습니다.
> * [WM-N400MSE_AT_Commands_Guide_v1.1][link-wm-n400mse-atcommand-manual]

### 1. Cat M1 모듈의 일반적인 동작 설명은 생략

>  Cat M1 모듈의 에코 모드 설명, USIM 상태 확인, 네트워크 접속 확인, PDP Context 활성화 등의 일반적인 내용은 TCP 가이드를 참고하시기 바랍니다.

### 2. 소켓 생성
소켓 서비스를 생성하는 명령어 입니다.

**AT Command:** AT+WSOCR

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | AT+WSOCR=(value1),(value2),(value3),(value4),(value5) | +WSOCR:(value6),(value7),(value8),(value9)<br><br>OK | AT+WSOCR=0,www.kma.go.kr,80,1,0<br>+WSOCR:1,0,64:ff9b::8b96:f9a2/80,TCP<br><br>OK |

**Defined values:**

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value1) | integer | Socket ID |
| (value2) | string | IP Address or URL |
| (value3) | integer | Port |
| (value4) | integer | Protocol<br>1 : TCP<br>2 : UDP |
| (value5) | integer | Packet Type<br>0 : ASCII<br>1 : HEX<br>2 : Binary |

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value6) | integer | Result<br>0 : 실패<br>1 : 성공 |
| (value7) | integer | Socket ID |
| (value8) | string | IP Adress/Port |
| (value9) | integer | Protocol<br>1 : TCP<br>2 : UDP |

### 3. 소켓 연결
지정된 소켓 서비스를 연결하는 명령어 입니다.

**AT Command:** AT+WSOCO

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | AT+WSOCO=(value1)| +WSOCO:(value2),(value3),OPEN_WAIT<br><br>OK<br>+WSOCO:(value4),OPEN_CMPL | AT+WSOCO=0<br>+WSOCO:1,0,OPEN_WAIT<br><br>OK<br>+WSOCO:0,OPEN_CMPL |

**Defined values:**

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value1) | integer | Socket ID |

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value2) | integer | Result<br>0 : 실패<br>1 : 성공 |
| (value3) | integer | Socket ID |
| (value4) | integer | Socket ID |

### 4. 소켓 데이터 전송

지정된 소켓으로 데이터를 전송하는 명령어 입니다.

**AT Command:** AT+WSOWR

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | AT+WSOWR=(value1),(value2),(value3) | +WSOWR:(value4),(value5)<br><br>OK | AT+WSOWR=0,93,GET /wid/queryDFSRSS.jsp?zone=4113552000 HTTP/1.1<br>HOST: www.kma.go.kr<br>Connection: close<br>(enter)<br><br>+WSOWR:1,0<br><br>OK |

**Defined values:**

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value1) | integer | Socket ID |
| (value2) | integer | Data Length |
| (value3) | string | Data |

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value4) | integer | Result<br>0 : 실패<br>1 : 성공 |
| (value5) | integer | Socket ID |

### 5. 소켓 데이터 수신
지정된 소켓으로부터 데이터를 수신하는 명령어 입니다.

**AT Command:** +WSORD

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Read | | +WSORD=(value1),(value2),(value3) (value4) OK<br>(value5) | +WSORD:0,1024,HTTP/1.1 200 OK<br>Date: Wed, 18 Sep 2019 00:45:41 GMT<br>Content-Length: 9174<br>Accept-Ranges: bytes<br>Content-Type: text/xml; charset=UTF-8<br>Connection: close<br><br><?xml version="1.0" encoding="UTF-8" ?><br>...

**Defined values:**

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value1) | integer | Socket ID |
| (value2) | integer | Buffer Size |
| (value3) | string | HTTP Version |
| (value4) | integer | Status Code |
| (value5) | string | Data |

### 6. 소켓 종료
지정된 소켓 서비스를 종료하는 명령어 입니다.

**AT Command:** AT+WSOCL

**Syntax:**

| Type | Syntax | Response | Example
|:--------|:--------|:--------|:--------|
| Write | AT+WSOCL=(value1) | +WSOCL:(value2),(value3),CLOSE_WAIT<br><br>OK<br>+WSOCL:(value4),CLOSE_CMPL | AT+WSOCL=0<br>+WSOCL:1,0,CLOSE_WAIT<br><br>OK<br>+WSOCL:0,CLOSE_CMPL |

**Defined values:**

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value1) | integer | Socket ID |

| Parameter | Type | Description |
|:--------|:--------|:--------|
| (value2) | integer | Result<br>0 : 실패<br>1 : 성공 |
| (value3) | integer | Socket ID |
| (value4) | integer | Socket ID |

<a name="Step-3-SampleCode"></a>
## 동작 구조 예제 (기상청 웹서버에 접속하여 분당구 날씨 확인)
```
// 소켓 생성
AT+WSOCR=0,www.kma.go.kr,80,1,0
+WSOCR:1,0,64:ff9b::8b96:f9a2/80,TCP

OK
// 소켓 연결
AT+WSOCO=0
+WSOCO:1,0,OPEN_WAIT

OK
+WSOCO:0,OPEN_CMPL
// 소켓 데이터 전송
AT+WSOWR=0,93,GET /wid/queryDFSRSS.jsp?zone=4113552000 HTTP/1.1
HOST: www.kma.go.kr
Connection: close


+WSOWR:1,0

OK
// 소켓 데이터 수신 (분당구 수내동의 기상 정보 확인)
+WSORD:0,1024,HTTP/1.1 200 OK
Date: Wed, 18 Sep 2019 00:45:41 GMT
Content-Length: 9174
Accept-Ranges: bytes
Content-Type: text/xml; charset=UTF-8
Connection: close

<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
<title>기상청 동네예보 웹서비스 - 경기도 성남시분당구 수내1동 도표예보</title>
<link>http://www.kma.go.kr/weather/main.jsp</link>
<description>동네예보 웹서비스</description>
<language>ko</language>
.....................................................................
<wfKor>맑음</wfKor>
.....................................................................
```


<a name="Step-4-Build-and-Run"></a>
## 예제 코드 빌드 및 실행

### 1. Import project
다음 링크에서 Arduino 예제 코드를 다운로드한 후, ino 확장자의 프로젝트 파일을 실행 시킵니다.

> 예제에서 활용할 Ping test sample code는 저장소의 아래 경로에 위치하고 있습니다.
> * `\samples\WIoT-WM01_WM-N400MSE\WIoT-WM01_Arduino_HTTP\`


### 2. Modify parameters

만약 다른 서비스의 API 테스트를 위해 URL을 변경하려는 경우, 다음 변수의 내용을 변경하면 됩니다.

````cpp
// Sample HTTP URL: Weather info by Korea Meteorological Administration
char request_url[] = "http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=4113552000";
````

### 3. Compile

상단 메뉴의 Compile 버튼을 클릭합니다.

컴파일이 완료 되면 다음과 같이 업로드를 수행하여 최종적으로 보드에 업로드를 수행 합니다.
업로드가 정상적으로 완료되면 'avrdude done. Thank you.' 메시지를 확인 할 수 있습니다.

![][compile2]

### 4. Run

예제 샘플 코드를 통해 Cat.M1 모듈의 HTTP client 운용 방법에 대해 파악할 수 있습니다.


#### 4.1 Connect your board
스타터 키트와 Arduino Mega2560과 Uart 통신을 하기위해서는 아래와 같이 점퍼 연결이 필요합니다.
예제 구동을 위해 WIZnet IoT Shield의 UART TXD와 RXD 핀을 Arduino Mega2560 보드의 'Serial 3' `TX3`(14), `RX3`(15) 에 연결합니다.

| ArduinoMega2560 | TX3 (14)  | RX3 (15) |
|:----:|:----:|:----:|
| WIZnet IoT Shield | RXD<br>(UART Rx for D1/D8)  | TXD<br>(UART Tx for D0/D2) |

> 보드 상단에 위치한 UART_SEL 점퍼를 제거한 후 (실크 기준) 오른쪽 핀을 Arduino 보드와 연결합니다.

![][hw-stack]

#### 4.2 Set up serial monitor
![][serialMonitor]

#### 4.3 Demo
![][1]
. . .
![][2]




[arduino-getting-started]: ./Arduino_get_started.md
[skt-iot-portal]: https://www.sktiot.com/iot/developer/guide/guide/catM1/menu_05/page_01
[link-arduino-compiler]: https://www.arduino.cc/en/Main/Software
[link-arduino Mega2560 Rev3]: https://store.arduino.cc/usa/mega-2560-r3
[link-bg96-atcommand-manual]: https://www.quectel.com/UploadImage/Downlad/Quectel_BG96_AT_Commands_Manual_V2.1.pdf
[link-bg96-mqtt-an]: https://www.quectel.com/UploadImage/Downlad/Quectel_BG96_MQTT_Application_Note_V1.0.pdf

[hw-stack]: ./imgs/hw/wiot-shield-wm01-arduinomega2560_stack.png 
[compile1]: ./imgs/arduino_guide_ide_compile.png
[compile2]: ./imgs/arduino_guide_ide_compile_finish.png
[serialMonitor]: ./imgs/arduino_guide_ide_serialmonitor.png

[1]: ./imgs/arduino_guide_wmn400_http-2.png
[2]: ./imgs/arduino_guide_wmn400_http-3.png
