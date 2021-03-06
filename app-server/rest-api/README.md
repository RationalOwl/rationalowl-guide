# REST API 개발 가이드

<!--
- [Introduction](#introduction)
- [앱서버 등록](#앱서버-등록)
  - [앱서버 등록 요청](#앱서버-등록-요청)
  - [앱서버 등록 결과](#앱서버-등록-결과)
- [앱서버 등록 해제](#앱서버-등록-해제)
  - [앱서버 등록 해제 요청](#앱서버-등록-해제-요청)
  - [앱서버 등록 해제 결과](#앱서버-등록-해제-결과)

- [단말그룹](#단말그룹)
- [단말그룹 생성](#단말그룹-생성)
  - [단말그룹 생성 요청](#단말그룹-생성-요청)
  - [단말그룹 생성 결과](#단말그룹-생성-결과)
- [단말그룹 내 단말 추가](#단말그룹-내-단말-추가)
  - [단말그룹 내 단말 추가 요청](#단말그룹-내-단말-추가-요청)
  - [단말그룹 내 단말 추가 결과](#단말그룹-내-단말-추가-결과)
- [단말그룹 내 단말 제거](#단말그룹-내-단말-제거)
  - [단말그룹 내 단말 제거 요청](#단말그룹-내-단말-제거-요청)
  - [단말그룹 내 단말 제거 결과](#단말그룹-내-단말-제거-결과)
- [단말그룹 삭제](#단말그룹-삭제)
  - [단말그룹 삭제 요청](#단말그룹-삭제-요청)
  - [단말그룹 삭제 결과](#단말그룹-삭제-결과)

- [다운스트림 메시지](#다운스트림-메시지)
  - [메시지 전송시간](#메시지-전송시간)
- [멀티캐스트](#멀티캐스트)
  - [멀티캐스트 메시지 발신](#멀티캐스트-메시지-발신)
  - [멀티캐스트 메시지 발신 결과](#멀티캐스트-메시지-발신-결과)
- [브로드캐스트](#브로드캐스트)
  - [브로드캐스트 메시지 발신](#브로드캐스트-메시지-발신)
  - [브로드캐스트 메시지 발신 결과](#브로드캐스트-메시지-발신-결과)
- [그룹메시지](#그룹메시지)
  - [그룹메시지 발신](#그룹메시지-발신)
  - [그룹메시지 발신 결과](#그룹메시지-발신-결과)

-->

## Introduction

> 래셔널아울 서비스를 이용하여 앱서버 개발하는 방법은 두가지가 있다. 래셔널아울 앱 서버 라이브러리를 이용하여 개발하는 방법과 REST API를 이용하는 방법이 그것이다.

앱서버 라이브러가 앱서버에 제공하는 주요 기능은 다음과 같다.    
     
#### 실시간 다운스트림 데이터 발신
     
현재 상용으로 서비스되는 푸시 메시지들은 다운스트림 데이터의 전송 속도, 전송률, 전송 순서 등 데이터 품질의 보장이 어려워 알림용도로만 이용된다. 래셔널아울 서비스는 다운스트림 데이터의 품질을 보장함으로써 다운스트림 데이터를 단순히 알림용도뿐 아니라 서비스 내 실시간 데이터 전달의 용도로 활용이 가능하다.

앱서버 라이브러리가 앱서버에 제공하는 다운스트림 발신 API는 다음과 같다.

- 멀티캐스트
- 브로드캐스트
- 그룹메시지

![이미지 이름](./img/downstream.png)      
     
#### 업스트림 데이터 수신
     
단말앱으로부터 업스트림 데이터 수신시 콜백을 제공한다.

![이미지 이름](./img/upstream.png)      
      
#### 단말 그룹 관리
      
단말 그룹은 단말을 그룹으로 묶는다. 단말 그룹에 포함되는 최대 단말 수는 백만이다. 즉, 1회의 그룹 메시지 발신으로 최대 백만 단말앱에 데이터를 전달 할 수 있다. 

앱서버 라이브러리에서 제공하는 단말 그룹 관리 API는 다음과 같다.

- 단말 그룹 생성
- 단말 그룹에 단말 추가
- 단말 그룹에서 단말 삭제
- 단말 그룹 삭제

REST API를 이용한 앱서버는 앱서버 라이브러리를 이용한 앱서버에 비해 아래와 같이 몇 가지 제한이 있다.

|         | 앱서버 라이브러리 이용 | REST API 이용|
|------------- | ------------- | ------|
|실시간 다운스트림 메시지 평균 전송 시간  | 0.1초 미만 | 0.2초 미만|
|업스트림 메시지 수신  | O  | X|
|콘솔에서 앱서버 상태 모니터링 지원 | O | X|

따라서 REST API는 앱 서버 라이브러리를 제공하지 않는 서버 개발 환경에 앱서버를 개발할 수 방안을 제공하는데 의의가 있다.

## 앱서버 등록

REST API를 이용하기 위해서 먼저 앱 서버를 등록한다. 하나의 모바일 서비스 내에서는 다수의 앱 서버 등록이 가능하다.
앱 서버가 등록되면 관리자 콘솔에 등록된 앱 서버가 나타난다. 다만 REST API로 등록된 앱 서버의 실시간 상태 모니터링은 지원되지 않는다.

![이미지 이름](./img/reg.png)

### 앱서버 등록 요청

주의할 점은 API호출시 전달하는 앱서버 등록이름과 등록결과 발급받은 앱서버 등록 아이디와 채널서버 URL을 반드시 저장/관리해야 한다. REST API를 통해 앱서버 등록이 성공했으면 해당 앱서버는 다시 등록 요청할 필요가 없다. 기등록한 앱서버가 다시 등록 요청할 경우 앱서버 등록 이름이 이전에 등록할 때와 다르면 래셔널아울 서비스에서는 다른 앱서버가 등록 요청하는 것으로 인식하여 앱서버를 새로 등록하게 된다.

통신 방식 : HTTP POST<br/>
url :   http://gate.rationalowl.com:8006/server/register/   
data format

```java
{
  "serviceId":"service id here",
  "registerName":"displayed app server name",

  // optional : 전용 에디션(구축형)에서 메시징 서버와 같은 private network에 앱서버가 운영될 경우 세팅.
  "privateNetwork":1
}
```

#### 필수 필드

- serviceId
  - 앱 서버가 등록할 서비스의 아이디 
  - service 등록 시 발급받는 아이디로 '관리자 콘솔 > 서비스 정보'에서 확인 가능
- registerName
  - 앱 서버를 콘솔에서 표시할 이름
  - 콘솔에서 앱서버를 확인하는 용도로 이용

#### 옵션 필드

- privateNetwork
  - 전용 에디션(구축형)에서 메시징 서버와 같은 private network에 앱서버가 운영될 경우 1 세팅
  - channelServer 값이 private network로 반환된다.

### 앱서버 등록 결과

```java
{
  "serverRegId":"asdfkjkjlasfdasfd",
  "channelServer":"211.211.223.44:9090"
}
```

#### 필수 필드

- serverRegId
  - 등록한 앱 서버의 아이디 
  - 이 후 REST API 호출시 이용되므로 앱 서버는 이를 저장해야 한다.
- channelServer
  - downstream 메시지 발신 시 이용할 채널서버
  - REST API URL의 (channelServer)에 입력해야 한다.

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지

## 앱서버 등록 해제

고객 서비스 내에서 사용하지 않는 앱 서버를 등록 해제한다.
등록 해제가 되면 앱 서버는 콘솔에서 사라진다.

### 앱서버 등록 해제 요청

통신 방식 : HTTP POST<br/>
url :   http://gate.rationalowl.com:8006/server/unregister/   
data format

```java
{
  "serviceId":"service id here",
  "serverRegId":"your app server registration ID"
}
```

#### 필수 필드

- serviceId
  - 앱 서버가 등록할 서비스의 아이디 
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디

### 앱서버 등록 해제 결과

```java
{
}
```

#### 필수 필드

없음

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지



## 단말그룹

>앱서버는 실시간 메시지를 주고 받기 위해 단말 앱 등록 아이디를 관리해야 한다. 즉, 단말앱도 앱스토어에서 앱을 다운받아 설치 후 최초 1회 단말 등록을 하고, 등록 결과 단말 등록 아이디를 발급받는다. 단말앱은 발급받은 단말 등록 아이디를 앱서버에게 upstream API를 통해 전달해주어야 하고 앱서버는 해당 단말 등록 아이디를 저장/관리해야 한다. 이후 단말로 다운스트림을 보낼때나 단말그룹을 관리할 때 단말 등록 아이디를 통해 관리하게 된다.

단말그룹은 그룹 메시지를 지원하기 위해서 생겨난 논리적인 단말들의 집합이다. 래셔널아울 서비스에서 제공하는 단말그룹의 특성은 다음과 같다.

- 단말그룹은 앱서버에서 생성/단말 추가/단말 삭제/그룹 삭제가 가능하다.
- 현재 그룹메시지는 다운스트림만 지원한다.
- 단말그룹에 추가될 수 있는 최대 단말수는 1,000,000이다.
- 단말그룹 생성시 지정할 수 있는 그룹내 단말 수는 2,000대이다.
- 단말그룹에 단말 추가시 한번에 추가할 수 있는 최대 단말 수는 2,000대이다.
- 래셔널아울 서비스에서 실시간 그룹메시지를 지원하는 최대 단말수는 2,000대이다.
- 실시간 메시지전달용으로 생성한 단말그룹내 단말 수는 2,000대가 넘지 않도록 해야한다.

앱서버 라이브러리에서 제공하는 단말 그룹 관리 API는 다음과 같다.

- 단말 그룹 생성
- 단말 그룹에 단말 추가
- 단말 그룹에서 단말 삭제
- 단말 그룹 삭제

단말 그룹 관리 API호출을 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.
래셔널아울 관리자 콘솔은 단말그룹 관리 API 호출 결과에 대해 실시간 모니터링을 제공한다.

## 단말그룹 생성

단말 그룹은 앱 서버가 생성 및 관리한다.
단말 그룹 생성시 그룹 내 단말 지정 대수는 최대 2000대이다.

### 단말그룹 생성 요청

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/deviceGroup/create/   
data format

```java
{
  //mandatory fields
  "serviceId":"service id here",
  "serverRegId":"server registration id here",
  "groupName":"group name",
  "deviceList":["deviceRegId1",......."deviceRegIdN"],
   //optional fields
  "description":"group description"
}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- groupName
  - 그룹명
  - 콘솔에서 디스플레이되는 이름
- deviceList
  - 단말 그룹에 포함될 단말 리스트
  - 최대 2천개까지 포함 가능
  - 2천개 이상 단말은 단말 그룹 생성 후 add device group을 통해 추가 가능.

#### 선택 필드

- description
  - 단말 그룹 설명 
  - 콘솔에서 디스플레이

### 단말그룹 생성 결과

```java
{
  "groupId":"asdfasfd",
  "deviceSize":2000,
  "failedDevice":["deviceRegId21",.."deviceRegId233"]
}
```

#### 필수 필드

- groupId
  - 등록한 그룹의 아이디
  - 이 후 그룹 메시지 발신시 이용되므로 앱 서버는 이를 저장해야 한다.
- deviceSize
  - 생성된 그룹에 포함된 단말 수
- failedDevice
  - 단말 그룹에 추가하려고 요청했으나 추가되지 않은 단말 목록
  - 현재는 기존 추가된 단말 등록 아이디를 다시 추가할 경우만 체크해서 반환
  - 앱서버 개발시 유효한 단말 등록 아이디 추가에 신경쓰야 함.

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지

## 단말그룹 내 단말 추가

생성된 단말 그룹에 단말을 추가한다.
한번에 추가할 수 있는 최대 단말 수는 2000개이다.
단말 그룹 추가 API를 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.

### 단말그룹 내 단말 추가 요청

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/deviceGroup/add/   
data format

```java
{
  //mandatory fields
  "serviceId":"service id here",
  "groupId":"adfaadfasdf",
  "deviceList":["deviceRegId1",......."deviceRegIdN"]
}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- groupId
  - 단말을 추가할 그룹의 그룹 아이디
- deviceList
  - 단말 그룹에 추가할 단말 리스트
  - 최대 2천개까지 포함 가능

### 단말그룹 내 단말 추가 결과

```java
{
  "groupId":"asdfasfd",
  "totaldeviceSize":22000,
  "addedDeviceSize":1800,
  "failedDevice":["deviceRegId21",.."deviceRegId233"]
}

```

#### 필수 필드

- groupId
  - 등록한 그룹의 아이디
  - 이 후 그룹 메시지 발신시 이용되므로 앱 서버는 이를 저장해야 한다.
- totalDeviceSize
  - API 호출 후 단말 그룹에 포함된 총 단말 수
- addedDeviceSize
  - API 호출로 단말 그룹에 새로 추가된 단말 수
- failedDevice
  - 단말 그룹에 추가하려고 요청했으나 추가되지 않은 단말 목록
  - 현재는 기존 추가된 단말 등록 아이디를 다시 추가할 경우만 체크해서 반환
  - 앱서버 개발시 유효한 단말 등록 아이디 추가에 신경쓰야 함.

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지

## 단말그룹 내 단말 제거

생성된 단말 그룹에서 단말을 제거한다.
한번에 제거할 수 있는 최대 단말 수는 2000개이다.
단말 그룹 관리 API를 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.

### 단말그룹 내 단말 제거 요청

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/deviceGroup/subtract/   
data format

```java
{
  //mandatory fields
  "serviceId":"service id here",
  "groupId":"adfaadfasdf",
  "deviceList":["deviceRegId1",......."deviceRegIdN"]
}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- groupId
  - 단말을 추가할 그룹의 그룹 아이디
- deviceList
  - 단말 그룹에서 삭제할 단말 리스트
  - 최대 2천개까지 포함 가능

### 단말그룹 내 단말 제거 결과

```java
{
  "groupId":"asdfasfd",
  "totaldeviceSize":22000,
  "removedDeviceSize":1800,
  "failedDevice":["deviceRegId21",.."deviceRegId233"]
}

```

#### 필수 필드

- groupId
  - 등록한 그룹의 아이디
  - 이 후 그룹 메시지 발신시 이용되므로 앱 서버는 이를 저장해야 한다.
- totalDeviceSize
  - API 호출 후 단말 그룹에 포함된 총 단말 수
- removedDeviceSize
  - API 호출로 단말 그룹에서 삭제된 단말 수
- failedDevice
  - 단말 그룹에 삭제하려고 요청했으나 삭제되지 않은 단말 목록
  - 앱서버 개발시 유효한 단말 등록 아이디에 신경 쓰야 함.

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지

## 단말그룹 삭제

단말그룹을 삭제한다.
단말그룹을 삭제하더라도 단말그룹 내 단말이 등록해제되는 것은 아니다.
단말 그룹 관리 API를 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.

### 단말그룹 삭제 요청

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/deviceGroup/remove/   
data format

```java
{
  //mandatory fields
  "serviceId":"service id here",
  "serverRegId":"server registration id here",
  "groupId":"adfaadfasdf"
}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- groupId
  - 삭제할 단말 그룹 아이디

### 단말그룹 삭제 결과

```java
{
}

```

#### 필수 필드

없음

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지



## 다운스트림 메시지

래셔널아울에서 앱서버에서 단말앱으로의 다운스트림 메시지는 기존 푸시 서비스의 단순 알림 용도가 실시간 데이터 전달의 용도로 이용한다. 그리고 래셔널아울 관리자 콘솔에서 제공하는 데이터 전달 현황 실시간 모니터링은 고객 서비스 개발시에는 개발의 용이함을 제공하고 서비스 운영시에는 서비스 대응력을 높이고 예측 가능성을 향상시킨다.

래셔널아울에서 지원하는 다운스트림 메시지의 특성은 다음과 같다.

- 기존 푸시 서비스에서 사용하던 푸시 알림 용도로 이용가능하다.
- 단말앱이 실행시에 평균 0.1초 미만의 실시간 데이터 전달의 용도로 이용한다.
- 지원하는 데이터 포맷은 스트링으로 일반 스트링문자나 json 포맷등 고객 서비스 특성에 맞게 설정하면 된다.
- 단말앱이 네트워크에 연결되지 않을 경우 큐잉 후 단말이 네트워크 접속시 전달하는 큐잉을 지원한다.
- 기본 큐잉 기간은 3일이고 전용 에디션에서는 최대 30일까지 설정 가능하다.
- 큐잉 기능을 이용할지 말지는 앱서버 라이브러리에서 제공하는 다운스트림 API에서 설정한다.
- 다운스트림 API에서 단말앱이 네트워크에 연결되지 않을 경우 전달 데이터외에 알림 타이틀과 알림 문자를 별도로 지정할 수 있다.
- 래셔널아울 콘솔은 데이터 전달 현황에 대해 실시간 모니터링을 제공한다.

### 메시지 전송시간

앱서버 라이브러리를 이용할 경우 다운스트림 메시지 전달시간은 평균 0.1초 미만이다. 반면 REST-API를 통한 다운스트림 메시지 전달시간은 0.2초 내외이다. 메시지 전송시간은 발신주체에서 메시지를 발신하여 수신주체가 메시지를 수신할 때까지 소요된 시간을 의미한다.

![이미지 이름](./img/delivery_time.png)

## 멀티캐스트

- 한대 이상의 단말앱에 메시지를 발신한다.
- 한번에 보낼 수 있는 최대 대상 단말 수는 2000대이다.

### 멀티캐스트 메시지 발신

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/downstream/multicast/   
data format

```java
{
  //mandatory fields
  "serviceId":"service id here",
  "serverRegId":"app server registration id",
  "deviceRegIds":["regId1","regId2",….,"regIdN],
  "queuing":1,
  "data":"{
    "numberData":11111,
    "stringData":"what you want",
     ...
  }",
  //optional fields
 "notiTitle":"notification title here",
 "notiBody":"notification body here",
 "notiSound":"my_custom_sound.wav"
}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- deviceRegIds
  - 메시지를 전달할 대상 단말의 단말 등록 아이디 목록
  - 단말앱이 최초 설치시 단말 앱 등록 시 발급받는 아이디로 발급 후
    앱 서버로 전달해 줘야 한다.
  - 앱 서버는 전달 받은 단말 등록 아이디를 저장/관리해야 한다.
  - 최대 2000대까지 제한
- queuing
  - 메시지 큐잉 지원 여부로 단말이 비 활성시 큐잉기간(기본 3일)동안 메시지를 래셔널아울서버에서 보관하고 있다가 단말이 활성 시 전달할지 여부
  - 1 : 지원
  - 0 : 미지원
- data
  - 단말앱에 전달할 데이터
  - 스트링으로 포맷은 json, 일반 스트링 고객 서비스 성격에 맞게 선택
  - 선택 필드인 noti 필드가 부재시 알림창에도 이용

#### 선택 필드

- notiTitle
  - 알림 제목 : 단말앱이 백그라운드/종료시 알림창에 표시할 제목
- notiBody
  - 알림 내용 : 단말앱이 백그라운드/종료시 알림창에 표시할 내용
- notiSound
  - 알림음 : 사용자 지정 알림음, 미설정시 기본 알림음이 울림

### 멀티캐스트 메시지 발신 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 발신한 메시지 아이디

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지

## 브로드캐스트

- 고객 서비스내에 등록된 모든 단말앱들에게 메시지를 발신한다.
- 브로드캐스트 메시지는 실시간 데이터 전달의 용도로 이용되어서는 안된다.
- 브로드캐스트 메시지는 전체 서비스내 단말앱에 푸시 알림 용도로만 사용해야 한다.

### 브로드캐스트 메시지 발신

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/downstream/broadcast/    
data format

```java
{
  //mandatory fields
  "serviceId":"service id here",
  "serverRegId":"app server registration id",
  "queuing":1,
  "data":"{
    "numberData":11111,
    "stringData":"what you want",
     ...
  }"
 //optional fields
 "notiTitle":"notification title here",
 "notiBody":"notification body here",
 "notiSound":"my_custom_sound.wav"

}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- queuing
  - 메시지 큐잉 지원 여부로 단말이 비 활성시 큐잉기간(기본 3일)동안 메시지를 래셔널아울서버에서 보관하고 있다가 단말이 활성 시 전달할지 여부
  - 1 : 지원
  - 0 : 미지원
- data
  - 단말앱에 전달할 데이터
  - 스트링으로 포맷은 json, 일반 스트링 고객 서비스 성격에 맞게 선택
  - 선택 필드인 noti 필드가 부재 시 알림 용도로도 이용

#### 선택 필드

- notiTitle
  - 알림 제목 : 단말앱이 백그라운드/종료시 알림창에 표시할 제목
- notiBody
  - 알림 내용 : 단말앱이 백그라운드/종료시 알림창에 표시할 내용
- notiSound
  - 알림음 : 사용자 지정 알림음, 미설정시 기본 알림음이 울림

### 브로드캐스트 메시지 발신 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 발신한 메시지 아이디

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 그룹메시지

그룹 메시지는 전통적인 통신방식인 멀티캐스트와 기능적으로 동일하다. 하지만 효율성, 속도, 대상 단말수 제한 등에서 그룹메시지가 훨씬 유리하다. 그룹메시지 내부 구현은 pub/sub모델로 확장성을 가지면서도 개발자에게는 전통적인 멀티캐스트의 장점인 명확한 대상 타게팅을 보장한다. 따라서 1회성 또는 단발성 통신을 제외하고는  멀티캐스트 API보다는 그룹메시지 API의 사용을 권고한다.

앱서버 라이브러리가 제공하는 전통적인 멀티캐스트API와 그룹메시지 API의 차이를 정리하면 다음과 같다.

|          | 그룹메시지 | 멀티캐스트 |
| ------------- | ------------- | ------|
| 최대 대상 단말수  | 1,000,000  | 2,000|
| 1회성/단발성 통신 | X | 권고|
| 1회성이 아닌 모든 통신| 권고 | X|

### 그룹메시지 발신

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/downstream/group/   
data format

```java
{	
  //mandatory fields
  "serviceId":"service id here",
  "serverRegId":"app server registration id",
  "groupId":"group id here",
  "queuing":1,
  "data":"{
    "numberData":11111,
    "stringData":"what you want",
     ...
  }"
 //optional fields
 "notiTitle":"notification title here",
 "notiBody":"notification body here",
 "notiSound":"my_custom_sound.wav"
}
```

#### 필수 필드

- serviceId
  - service 등록 시 발급받는 아이디로 콘솔 서비스 정보에서 확인 가능
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- groupId
  - 단말 그룹 아이디
- queuing
  - 메시지 큐잉 지원 여부로 단말이 비 활성시 큐잉기간(기본 3일)동안 
    메시지를 래셔널아울서버에서 보관하고 있다가 단말이 활성 시 전달할지 여부
  - 1 : 지원
  - 0 : 미지원
- data
  - 단말앱에 전달할 데이터
  - 스트링으로 포맷은 json, 일반 스트링 고객 서비스 성격에 맞게 선택
  - 선택 필드인 noti 필드가 부재 시 알림 용도로도 이용

#### 선택 필드

- notiTitle
  - 알림 제목 : 단말앱이 백그라운드/종료시 알림창에 표시할 제목
- notiBody
  - 알림 내용 : 단말앱이 백그라운드/종료시 알림창에 표시할 내용
- notiSound
  - 알림음 : 사용자 지정 알림음, 미설정시 기본 알림음이 울림

### 그룹메시지 발신 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 발신한 메시지 아이디

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 커스텀푸시

이미지 푸시와 팝업 푸시 등 앱 성격에 맞게 푸시알림의 형태를 임의로 표현할 수 있다. 리치 노티피케이션(Rich Notification)이라고도 회자되는데 커스텀푸시(Customizable Push)라는 용어가 가장 적절할 것 같다. 래셔널아울 서비스에서는 이를 커스텀 푸시라는 용어로 통일하여 사용한다. 커스텀푸시는 기존 리치 노티피케이션에 실시간 트래킹과 모니터링 기능을 추가한 개념이다.

커스텀 푸시의 전반적인 개념은 아래 서비스 블로그 문서를 참조하는 것이 효과적이다.

- [커스텀푸시 개념](https://rationalowl.tistory.com/20) 


## 커스텀푸시 멀티캐스트

- 한대 이상의 단말앱에 커스텀푸시 메시지를 발신한다.
- 한번에 보낼 수 있는 최대 대상 단말 수는 2000대이다.

### 커스텀푸시 멀티캐스트 메시지 발신

통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/custompush/multicast/ </br>
data format

```java
{
   //mandantory fields
  "serviceId":"service id here",
  "serverRegId":"server registration id here",
  "deviceRegIds":[“regId1“,”regId2”,….,”regIdN],
  "silent":0,
  "data":{
     // your custom field & data
     // add whatever fields you want.
    "numberData":11111,
    "stringData":"what you want",     
    "notiTitle":"notification title",
    "notiBody":"notification data",
    "notiSound":"custom_sound.wav",
    "imgUrl":"myurl/myimag.jpg",
    "soundUrl":"myurl/myimag.wav"
  }
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- deviceRegIds
  - 메시지를 전달할 대상 단말의 단말 등록 아이디 목록
  - 단말앱이 최초 설치시 단말 앱 등록 시 발급받는 아이디로 발급 후
    앱 서버로 전달해 줘야 한다.
  - 앱 서버는 전달 받은 단말 등록 아이디를 저장/관리해야 한다.
  - 최대 2000대까지 제한
- silent
  - 0: 일반 커스텀푸시
  - 1: 사일런트 커스텀푸시
- data
  - 사용자 지정 데이터 필드 및 값
  - 앱 성격에 맞게 임의의 필드 추가 가능

#### 선택 필드

- NA

### 커스텀푸시 멀티캐스트 메시지 발신 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 발신한 메시지 아이디
  - 재발신 및 메시지 전달 트래킹 시 사용됨

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 커스텀푸시 브로드캐스트


### 커스텀푸시 브로드캐스트 메시지 발신


모든 단말앱에 메시지를 발신한다.<br/>


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/custompush/broadcast/ </br>
data format

```java
{
   //mandantory fields
  "serviceId":"service id here",
  "serverRegId":"server registration id here",
  "silent":0,
  "data":{
     // your custom field & data
     // add whatever fields you want.
    "numberData":11111,
    "stringData":"what you want",     
    "notiTitle":"notification title",
    "notiBody":"notification data",
    "notiSound":"custom_sound.wav",
    "imgUrl":"myurl/myimag.jpg",
    "soundUrl":"myurl/myimag.wav"
  }
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- silent
  - 0: 일반 커스텀푸시
  - 1: 사일런트 커스텀푸시
- data
  - 사용자 지정 데이터 필드 및 값
  - 앱 성격에 맞게 임의의 필드 추가 가능

#### 선택 필드

- NA

### 커스텀푸시 브로드캐스트 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 발신한 메시지 아이디
  - 재발신 및 메시지 전달 트래킹 시 사용됨

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 커스텀푸시 그룹메시지


### 커스텀푸시 그룹메시지 발신


단말그룹에 속한 단말에게 메시지를 발신한다.<br/>


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/custompush/group/ <br/>
data format

```java
{
   //mandantory fields
  "serviceId":"service id here",
  "serverRegId":"server registration id here",
  "groupId":"group id here",
  "silent":0,
  "data":{
     // your custom field & data
     // add whatever fields you want.
    "numberData":11111,
    "stringData":"what you want",     
    "notiTitle":"notification title",
    "notiBody":"notification data",
    "notiSound":"custom_sound.wav",
    "imgUrl":"myurl/myimag.jpg",
    "soundUrl":"myurl/myimag.wav"
  }
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- groupId
  - 커스텀 푸시를 전달할 대상 단말그룹의 아이디
- silent
  - 0: 일반 커스텀푸시
  - 1: 사일런트 커스텀푸시  
- data
  - 사용자 지정 데이터 필드 및 값
  - 앱 성격에 맞게 임의의 필드 추가 가능

#### 선택 필드

- NA

### 커스텀푸시 그룹메시지 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 발신한 메시지 아이디
  - 재발신 및 메시지 전달 트래킹 시 사용됨

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 커스텀 푸시 재발신

 앞서 위의 세 커스텀 푸시 발신 API로 발신 후 알림 미전달 혹은 알림수신 미확인 대상 사용자 폰에 재발신한다.

### 커스텀 푸시 재발신


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/custompush/retry/<br/>
data format

```java
{
   //mandantory fields
  "serviceId":"service id here",
  "serverRegId":"app server registration id",
  "msgId":"msgId",
  "retryCondition":1
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- msgId
  - 재전달할 커스텀푸시의 메시지 아이디
- retryCondition
  - 재발신 조건
    - 1: 알림미전달 폰에만 재발신
    - 2: 수신미확인 폰에만 재발신

#### 선택 필드

- NA

### 커스텀 푸시 재발신 결과

```java
{
    "mId":"message id here"
}
```

#### 필수 필드

- mId
  - 재발신한 메시지 아이디  

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지



## 메시지 전달 트래킹(메시지 통계 정보)

  한번에 다수의 메시지 전달통계를 실시간 확인한다. 
  - 커스텀푸시 및 실시간 메시지 모두 지원한다.
  - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.   

### 메시지 전달 트래킹(메시지 통계 정보) API


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/tracking/msgStats/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"server1",
  "msgIds":["ddddd","ddssss"]
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- msgIds
  - 전달통계를 알고자하는 커스텀 푸시의 메시지 아이디 목록
  - 한번에 최대 100개 메시지 아이디 지정


#### 선택 필드

- NA

### 메시지 전달 트래킹(메시지 통계 정보) 결과

```java
{
    "serviceId":"0909",
    "serverRegId":"server1",
     "msgs":
      [
        {"msgId":"090909","total":232222,"que":9494,"notiDeliver":111,"deliver":555,"expire":110},
        {"msgId":"090911","total":232222,"que":9494,"notiDeliver":111,"deliver":555,"expire":110},….
      ]
      //실패시 error 필드 세팅..
}
```

#### 필수 필드

- serviceId
  - 서비스 구분자
- msgs: 각 메시지 별 전달 현황 정보들의 배열
  - msgId: 커스텀푸시의 메시지 아이디
  - total: 커스텀푸시를 발신한 대상 전체 사용자 단말 수
  - que: 커스텀푸시를 전달하지 못한 대상 사용자 단말 수
  - notiDeliver: 커스텀푸시은 전달하였으나 사용자가 알림확인은 하지 않은 사용자 단말 수
  - deliver: 커스텀푸시를 사용자가 확인한 사용자 단말 수
  - expire: 래셔널아울 메시지 큐잉 기간동안 커스텀푸시를 전달하지 못하여 기간만료가 된 사용자 단말 수

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 메시지 전달 트래킹(메시지 상태별 단말앱 목록)

 하나의 커스텀푸시 메시지 상태별 단말앱 목록을 실시간 트래킹한다.
 - 알림 미전달 단말앱 목록만 트래킹하여 알림 미전달 단말앱 대상 유료 메시지 발신시 유용하다.   

 - 멀티캐스트, 그룹메시지, 브로드캐스트로 동시에 1000단말앱이상 발신한 메시지의 트래킹에 효과적이다.

 - 100만 단말앱에 동시에 커스텀 푸시 발신시에도 1분대에 전체 발신 및 전달 및 수신확인 여부 실시간 트래킹이 가능하다.

 - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.   

### 메시지 전달 트래킹(메시지 상태별 단말앱 목록) API


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/tracking/msgDevices/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"server1",
  "msgId":"ddddd",
  "msgState":1,  
  "deviceType": 1,
  "startIndex":0,
  "fetchSize":500
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- msgId: 커스텀 푸시의 메시지 아이디
- msgState: 메시지 상태 플래그
  - 0: 모든 단말 목록
  - 1: 푸시알림 미전달 단말 목록
  - 2: 푸시알림 수신확인한 단말 목록
  - 4: 푸시알림 미전달 기간만료한 단말 목록
  - 8: 푸시알림은 전달했으나 수신확인하지 않은 단말 목록
  - bit or 연산으로 지정 가능 ex) 7 = 1 | 2 | 4 
- deviceType: 단말 타입 플래그
  - 0: 모든 단말 목록
  - 1: 안드로이드 단말 목록
  - 2: IOS 단말 목록
  - bit or 연산으로 지정 가능
- startIndex: 정보를 알고자 하는 단말 목록의 처음 인덱스
  - '4. 커스텀푸시 전달 통계 요청' REST API 결과로 확인한 대상 단말 수가 1000대 이상일 경우 본 REST API를 여러번 호출해야 한다. ex) 대상 단말이 1만대일 경우 fetchSize를 1000개씩 10번 호출해야 하고 startIndex는 0, 1000, 2000, ... 순으로 증가하면서 호출해야 한다.
- fetchSize: 한번에 가지고 올 단말정보 목록 갯수
  - 한번에 최대 1000까지 지원


#### 선택 필드

- NA

### 메시지 전달 트래킹(메시지 상태별 단말앱 목록) 결과

```java
{
    "serviceId":"0909",
     "msgId":"dddd",
     "startIndex":0,
     "totalDeviceSize":10000,
     "fetchDeviceSize":1000,
     "deviceList":["deviceId1","deviceId2",....."deviceId1000"]
      

      //실패시 error 필드 세팅..
}
```

#### 필수 필드

- serviceId: 서비스 구분자
- startIndex:  총 단말 목록 중 fetch할 첫번째 인덱스
- totalDeviceSize: 조건에 부합하는 전체 대상 단말 수
- fetchDeviceSize: 결과로 전달받은 단말정보 목록 수
- deviceList: 요청한 조건에 부합하는 단말 아이디 목록 

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지

## 메시지 전달 트래킹(메시지 상태별 단말앱 상세정보)

 하나의 커스텀푸시 메시지 상태별 단말앱 상세정보를 실시간 트래킹한다.
   - 멀티캐스트, 그룹메시지, 브로드캐스트로 동시에 1000단말앱이상 발신한 메시지의 트래킹에 효과적이다.
   - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.   

### 메시지 전달 트래킹(메시지 상태별 단말앱 상세정보) API


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/tracking/msgDeviceDetail/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"server1",
  "msgId":"ddddd",
  "msgState":1,  
  "deviceType": 1,
  "startIndex":0,
  "fetchSize":500
}
```

#### 필수 필드

- serviceId
  - 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId
  - 등록 시 발급받은 앱 서버 등록 아이디
- msgId: 커스텀 푸시의 메시지 아이디
- msgState: 메시지 상태 플래그
  - 0: 모든 단말 목록
  - 1: 푸시알림 미전달 단말 목록
  - 2: 푸시알림 수신확인한 단말 목록
  - 4: 푸시알림 미전달 기간만료한 단말 목록
  - 8: 푸시알림은 전달했으나 수신확인하지 않은 단말 목록
  - bit or 연산으로 지정 가능 ex) 7 = 1 | 2 | 4 
- deviceType: 단말 타입 플래그
  - 0: 모든 단말 목록
  - 1: 안드로이드 단말 목록
  - 2: IOS 단말 목록
  - bit or 연산으로 지정 가능
- startIndex: 정보를 알고자 하는 단말 목록의 처음 인덱스
  - '4. 커스텀푸시 전달 통계 요청' REST API 결과로 확인한 대상 단말 수가 1000대 이상일 경우 본 REST API를 여러번 호출해야 한다. ex) 대상 단말이 1만대일 경우 fetchSize를 1000개씩 10번 호출해야 하고 startIndex는 0, 1000, 2000, ... 순으로 증가하면서 호출해야 한다.
- fetchSize: 한번에 가지고 올 단말정보 목록 갯수
  - 한번에 최대 1000까지 지원


#### 선택 필드

- NA

### 메시지 전달 트래킹(메시지 상태별 단말앱 상세정보) 결과

```java
{
    "serviceId":"0909",
     "msgId":"dddd",
     "startIndex":0,
     "totalDeviceSize":10000,
     "fetchDeviceSize":1000,
     "deviceList":[
       {"deviceRegId":"dkdkd","deviceRegName":"david's phone","os":1,"deviceState":1,"msgState":1,"deliverTime":11111,"notiTime":222222},
        ......
      ]
      

      //실패시 error 필드 세팅..
}
```

#### 필수 필드

- serviceId: 서비스 구분자
- startIndex:  총 단말 목록 중 fetch할 첫번째 인덱스
- totalDeviceSize: 조건에 부합하는 전체 대상 단말 수
- fetchDeviceSize: 결과로 전달받은 단말정보 목록 수
- deviceList: 전달받은 단말정보 목록
  - deviceRegId: 단말등록 아이디
  - deviceRegName: 사용자가 등록한 단말앱 구분자(웹 관리자 화면에 표시되는 사용자가 읽고 이해할수 있도록 설정해야 함
  - os: 1(안드로이드), 2(IOS)
  - msgState: 1(미전달), 2(사용자 수신 확인), 3(미전달 기간만료), 4(푸시알림 전달)
  - deliverTime: 메시지수신확인시간
    - msgState가 2일 경우만 세팅
    - 사용자가 메시지를 확인한 시간 (UTC 1970/1/1/0/0/0 으로부터 경과한 밀리세컨드)
  - notiTime: 푸시알림 전달시간
    - msgState가 4인 경우 세팅(푸시 알림만 전달)
    - msgState가 2인 경우는 세팅될 수도 안될 수도 있다.
      - 푸시알림을 수신한 후 사용자 수신확인시 알림전달 시간 세팅
      - 사용자가 단말앱을 실행하여 푸시알림을 전달하지 않고 실시간으로 메시지를 전달시 알림전달 시간 미세팅 

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 배치 메시지 전달 트래킹(메시지 상태별 단말앱 목록)

 한번에 다수의 메시지를 트래킹시 사용한다.
   - 커스텀푸시 메시지 상태별 단말앱 목록을 실시간 트래킹한다.
   - 개별로 전달하는 개인화 메시지나 대상 단말앱이 1000미만인 메시지인 경우 동시에 여러 메시지에 대한 트래킹시 이용한다.
   - 한번에 최대 100메시지 트래킹 지원
   - 전체 메시지의 대상 단말앱 수가 1000미만이어야 한다.
   - 알림 미전달 단말앱 목록만 트래킹하여 알림 미전달 단말앱 대상 유료 메시지 발신시 유용하다.
   - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.      

### 배치 메시지 전달 트래킹(메시지 상태별 단말앱 목록) API


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/tracking/batchMsgDevices/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"server1",
  "msgIds":["aaaaaa", "bbbbb", "cccccc"],
  "msgState":1
}
```

#### 필수 필드

- serviceId: 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId: 트래킹 요청하는 서버 자신의 서버등록 아이디
- msgIds: 커스텀 푸시의 메시지 아이디 목록
- msgState: 메시지 상태 플래그
  - 0: 모든 단말 목록
  - 1: 푸시알림 미전달 단말 목록
  - 2: 푸시알림 수신확인한 단말 목록
  - 4: 푸시알림 미전달 기간만료한 단말 목록
  - 8: 푸시알림은 전달했으나 수신확인하지 않은 단말 목록
  - bit or 연산으로 지정 가능 ex) 7 = 1 | 2 | 4 


#### 선택 필드

- NA

### 배치 메시지 전달 트래킹(메시지 상태별 단말앱 목록) 결과

```java
{
    "serviceId":"0909",
    "msgs":
       [
       	 {"msgId":"aaaaaa","deviceList":["111111","222222","333333"]},
         {"msgId":"bbbbbb","deviceList":["111111","222222","333333"]},
         ....
         {"msgId":"dddddd","deviceList":["111111","222222","333333"]}         
       ]      

      //실패시 error 필드 세팅..
}
```

#### 필수 필드

- serviceId: 서비스 구분자
- msgs: 배치 메시지별 트래킹 결과
  - msgId: 메시지 아이디
  - deviceList: 요청 조건에 부합하는 단말아이디 목록

#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지


## 배치 메시지 전달 트래킹(메시지 상태별 단말앱 상세정보)

 한번에 다수의 메시지를 트래킹시 사용한다.
   - 커스텀푸시 메시지 상태별 단말앱 상세정보를 실시간 트래킹한다.
   - 개별로 전달하는 개인화 메시지나 대상 단말앱이 1000미만인 메시지인 경우 동시에 여러 메시지에 대한 트래킹시 이용한다.
   - 한번에 최대 100메시지 트래킹 지원
   - 전체 메시지의 대상 단말앱 수가 1000미만이어야 한다.
   - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.

### 배치 메시지 전달 트래킹(메시지 상태별 단말앱 상세정보) API


통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/tracking/batchMsgDeviceDetail/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"server1",
  "msgIds":["aaaaaa", "bbbbb", "cccccc"],
  "msgState":1
}
```

#### 필수 필드

- serviceId: 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId: 트래킹 요청하는 서버 자신의 서버등록 아이디
- msgIds: 커스텀 푸시의 메시지 아이디 목록
- msgState: 메시지 상태 플래그
  - 0: 모든 단말 목록
  - 1: 푸시알림 미전달 단말 목록
  - 2: 푸시알림 수신확인한 단말 목록
  - 4: 푸시알림 미전달 기간만료한 단말 목록
  - 8: 푸시알림은 전달했으나 수신확인하지 않은 단말 목록
  - bit or 연산으로 지정 가능 ex) 7 = 1 | 2 | 4 


#### 선택 필드

- NA

### 배치 메시지 전달 트래킹(메시지 상태별 단말앱 상세정보) 결과

```java
{
    "serviceId":"0909",
    "msgs":
    [
       {
         "msgId":"aaaaaa",
         "deviceList":
         [
           {"deviceRegId":"dkdkdk","deviceRegName":"david's phone","os":1,"msgState":1,"deliverTime":11111},
           {"deviceRegId":"dkdkdk","deviceRegName":"david's phone","os":1,"msgState":1,"deliverTime":11111},
           ...
           {"deviceRegId":"dkdkdk","deviceRegName":"david's phone","os":1,"msgState":1,"deliverTime":11111}
         ]  
       },
       {
          "msgId":"bbbbb"
          ....
       }
        ......
    ]     

    //실패시 error 필드 세팅..
}
```

#### 필수 필드

- serviceId: 서비스 구분자
- msgs: 배치 메시지별 트래킹 결과
  - msgId: 메시지 아이디
  - deviceList: 요청 조건에 부합하는 단말앱 상세정보 목록
    - deviceRegId: 단말등록 아이디
    - deviceRegName: 사용자가 등록한 단말앱 구분자(웹 관리자 화면에 표시되는 사용자가 읽고 이해할수 있도록 설정해야 함
    - os: 1(안드로이드), 2(IOS)
    - msgState: 1(미전달), 2(사용자 수신 확인), 3(미전달 기간만료), 4(푸시알림 전달)
    - deliverTime: 메시지수신확인시간
      - msgState가 2일 경우만 세팅
      - 사용자가 메시지를 확인한 시간 (UTC 1970/1/1/0/0/0 으로부터 경과한 밀리세컨드)
    - notiTime: 푸시알림 전달시간
      - msgState가 4인 경우 세팅(푸시 알림만 전달)
      - msgState가 2인 경우는 세팅될 수도 안될 수도 있다.
        - 푸시알림을 수신한 후 사용자 수신확인시 알림전달 시간 세팅
        - 사용자가 단말앱을 실행하여 푸시알림을 전달하지 않고 실시간으로 메시지를 전달시 알림전달 시간 미세팅
#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지



>## 단말앱 동기화

앱서버는 등록한 단말앱 아이디을 관리해야 한다. 가장 기본적인 관리방법이 단말앱이 단말앱 등록 API와 단말앱 등록해제 API 호출 후 그 결과를 앱서버에 전달함으로써 앱서버가 유효한 단말앱 아이디를 동기화하는 것이다. 그러나 사용자가 단말앱 삭제를 하는 경우 앱서버는 삭제된 단말앱 아이디를 알기가 어렵다. 이를 해결하기 위해 래셔널아울은 두 가지 방법의 단말앱 동기화 방안을 제공한다.

 첫째, 자동 동기화 방식인데 래셔널아울 서비스 차원에서 삭제한 단말앱을 추적하여 단말앱 상태를 동기화하고 있다가 앱서버가 동기화 API 호출을 통해 동기화하는 방식이다. 이는 래셔널아울 웹관리자 화면의 '단말앱 현황' 화면에서 '단말앱 시스템푸시 설정'메뉴를 통해 시스템 푸시를 '허용'으로 설정하면 그때부터 래셔널아울은 해당 서비스의 전체 단말앱 삭제여부를 트래킹하고 동기화한다. 

  시스템 푸시는 래셔널아울 서비스 차원에서 단말앱에 발신하는 사일런트 푸시이다. 시스템 푸시를 허용으로 설정하면 해당 서비스 내 모든 단말앱에 하루 2번 시스템을 발신하고 모든 단말앱들의 시스템 푸시 수신 여부를 트래킹한다. 참고로 시스템 푸시는 통계에 수치가 반영되지 않고 과금이 되지 않는다.

 자동 동기화 방식에서 래셔널아울이 단말앱 삭제 판단 기준은 아래 두 가지 조건을 모두 만족하는 경우이다.
 - 단말앱이 10일 이상 시스템푸시를 수신하지 않음
 - 단말앱이 10일 이상 실행되지 않음
 
단말앱 삭제 판단 기준일 10일은 구축형의 경우 고객사가 설정 가능하나 클라우드 형의 경우 설정을 지원하지 않는다.

둘째, 개발사 직접 수행하는 수동 동기화 방식이다. 서비스 성격에 따라 단말앱 삭제 판단 기준일을 3일로 빠른 동기화가 필요할 수도 있고 한달 정도로 판단기준일을 길게 잡고자 할 경우에 적합하다. 

수동 동기화 방식은 자동화 방식에서 래셔널아울이 시스템푸시를 통해 동기화하는 방식을 직접 개발사에서 진행하면 된다. 즉, 고객사 앱서버에서 매일 하루 한 번 이상 사일런트 커스텀푸시를 모든 단말앱에 발신하여 메시지 트래킹 API 혹은 자동 메시지 트래킹 모드의 콜백을 통해 모든 단말앱의 알림전달 여부를 확인하여 삭제 판단 기준일 초과 단말앱아이디를 삭제 처리하면 된다. 직접 동기화 방식에서 주의할 점은 시스템푸시를 불허로 한 경우 래셔널아울은 삭제된 단말앱에 대한 동기화를 진행하지 않기 때문에 고객사 앱서버에서 unregisterDeviceIds API를 통해 사용자 삭제 단말앱 아이디 목록을 래셔널아울에 알려줘야 한다는 것이다. 

 경우에 따라 자동/수동 동기화 방식을 같이 사용할 수 도 있다.


## 유효한 단말앱 아이디 목록 조회 
 시스템푸시를 허용으로 단말앱 자동 동기화 방식을 이용하는 경우 래셔널아울은 삭제된 단말앱까지 동기화하게 된다. 앱서버는 주기적으로 유효한 단말앱 아이디 목록 조회 API를 호출하면 서비스 내 모든 유효한 단말앱 목록을 동기화해야 한다. 한번의 API호출로 최대 2,000개 단말앱 아이디를 조회할 수 있다. 따라서 100만개 단말앱이 있는 서비스의 경우 해당 API를 500회 호출해야 한다.
   
통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/sync/deviceIds/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"dkdkdkdk",
  "startIndex":0,
  "fetchSize":2000
}
```

#### 필수 필드

- serviceId: 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId: 앱서버 자신의 서버등록 아이디
- startIndex: 전체 단말앱 중 fetch 시작 index
- fetchSize: 전체 단말앱 목록 startIndex부터 fetch할 단말앱 아이디 목록 수, 최대값 2,000


#### 선택 필드

- NA

###  결과

```java
{
  "totalDeviceSize":3,
  "startingIndex":0,
  "fetchDeviceSize":3,
  "deviceList":[
    "11111","222222","333333"
  ]      
  //실패시 error 필드 세팅..
}
```

#### 필수 필드
- totalDeviceSize: 서비스 내 유효한 단말앱 아이디 전체 갯수
- startIndex: 전체 단말앱 중 fetch 시작 index
- fetchSize: deviceList 내 단말앱 아이디 갯수
- deviceList: 실제 fetch한 단말앱 아이디 목록
#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지



## 단말그룹 내 단말앱 아이디 목록 조회 
 자동 동기화 방식을 이용하는 경우 앱서버는 주기적으로 단말그룹 내 단말앱 아이디 목록 조회 API를 호출하여 단말그룹 내 모든 유효한 단말앱 목록을 동기화해야 한다. 한번의 API호출로 최대 2,000개 단말앱 아이디를 조회할 수 있다. 따라서 100만개 단말앱이 있는 단말그룹의 경우 해당 API를 500회 호출해야 한다.
   
단말앱 수동 동기화 방식에서 삭제 판단일 초과 단말앱에 댛새 앱서버는 /sync/deleteDevices/ API를 호출하여 래셔널아울 서비스에서도 동기화 하도록 알려야 한다.
  
통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/sync/groupDeviceIds/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"dkdkdkdk",
  "groupId":"dddddd",
  "startIndex":0,
  "fetchSize":2000
}
```

#### 필수 필드

- serviceId: 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId: 앱서버 자신의 서버등록 아이디
- groupId: 대상 단말그룹 아이디
- startIndex: 전체 단말앱 중 fetch 시작 index
- fetchSize: 전체 단말앱 목록 startIndex부터 fetch할 단말앱 아이디 목록 수, 최대값 2,000


#### 선택 필드

- NA

###  결과

```java
{
  "groupId":"dddddd",
  "totalDeviceSize":3,
  "startingIndex":0,
  "fetchDeviceSize":3,
  "deviceList":[
    "11111","222222","333333"
  ]      
  //실패시 error 필드 세팅..
}
```

#### 필수 필드
- groupId: 대상 단말그룹 아이디
- totalDeviceSize: 단말그룹 내 유효한 단말앱 아이디 전체 갯수
- startIndex: 전체 단말앱 중 fetch 시작 index
- fetchSize: deviceList 내 단말앱 아이디 갯수
- deviceList: 실제 fetch한 단말앱 아이디 목록
#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지



## 단말앱 등록 해제 
 단말앱 수동 동기화 방식에서 삭제 판단일 초과 단말앱에 댛새 앱서버는 /sync/deleteDevices/ API를 호출하여 래셔널아울 서비스에서도 동기화 하도록 알려야 한다.
  
통신 방식 : HTTP POST<br/>
url :   http://221.221.22.11:8006(channelServer)/sync/deleteDevices/<br/>
data format

```java
{
  //mandatory fields
  "serviceId":"service1",
  "serverRegId":"server1",
  "deviceIds":["1111111","2222222"]
}
```

#### 필수 필드

- serviceId: 래셔널아울 콘솔에 등록한 서비스 구분자
- serverRegId: 앱서버 자신의 서버등록 아이디
- deviceIds: 래셔널아울에서 등록해제할 단말앱 아이디 목록


#### 선택 필드

- NA

###  결과

```java
{
   //실패시 error 필드 세팅..
}
```

#### 필수 필드
- 단말앱 아이디 목록이 정상적으로 등록해제된 경우 필드 없음
#### 선택 필드

- error
  - REST API 실패 시 반환
  - 실패 원인 메시지
