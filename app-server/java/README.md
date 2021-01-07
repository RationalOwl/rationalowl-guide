# 래셔널아울 JAVA 앱서버 개발 가이드

>## Introduction

> 앱서버는 앱서버 라이브러리가 제공하는 API를 통해 단말 앱들과 실시간 데이터 통신을 할 수 있다. 


앱서버 라이브러가 앱서버에 제공하는 주요 기능은 다음과 같다.

1. 실시간 다운스트림 데이터 발신
현재 상용으로 서비스되는 푸시 메시지들은 다운스트림 데이터의 전송 속도, 전송률, 전송 순서 보장로 대표되는 데이터 품질의 보장이 어려워 알림용도로만 이용된다. 래셔널아울 서비스는 다운스트림 데이터의 품질을 보장함으로써 다운스트림 데이터를 단순히 알림용도뿐 아니라 서비스 내 실시간 데이터 전달의 용도로 활용이 가능하다.

앱서버 라이브러리가 앱서버에 제공하는 다운스트림 발신 API는 다음과 같다.
- 멀티캐스트
- 브로드캐스트
- 그룹메시지

![이미지 이름](./img/downstream.png)

2. 업스트림 데이터 수신시 콜백
단말앱으로부터 업스트림 데이터 수신시 콜백을 제공한다.

![이미지 이름](./img/upstream.png)

3. 단말 그룹 관리
단말 그룹은 단말을 그룹으로 묶는다. 단말 그룹에 포함되는 최대 단말 수는 백만이다. 즉, 1회의 그룹 메시지 발신으로 최대 백만 단말앱에 데이터를 전달 할 수 있다. 

앱서버 라이브러리에서 제공하는 단말 그룹 관리 API는 다음과 같다.
  - 단말 그룹 생성
  - 단말 그룹에 단말 추가
  - 단말 그룹에서 단말 삭제
  - 단말 그룹 삭제

4. 멀티캐스트 VS 그룹 메시지

그룹 메시지는 전통적인 통신방식인 멀티캐스트와 기능적으로 동일하다. 하지만 효율성, 속도, 대상 단말수 제한 등에서 그룹메시지가 훨씬 유리하다. 그룹메시지 내부 구현은 pub/sub모델로 확장성을 가지면서도 개발자에게는 전통적인 멀티캐스트의 장점인 명확한 대상 타게팅을 보장한다. 따라서 1회성 또는 단발성 통신을 제외하고는  멀티캐스트 API보다는 그룹메시지 API의 사용을 권고한다.

앱서버 라이브러리가 제공하는 전통적인 멀티캐스트API와 그룹메시지 API의 차이를 정리하면 다음과 같다.

|             | 그룹메시지 | 멀티캐스트
------------- | ------------- | ------
최대 대상 단말수  | 1,000,000  | 2,000
평균 데이터 전송속도  | 0.08초  | 009초
1회성/단발성 통신 | X | 권고
1회성이 아닌 모든 통신| 권고 | X





>##  앱 서버 라이브러리 적용

JAVA 앱서버 개발을 위한 설정은 JAVA 앱서버 라이브러리를 개발 환경에 포함하는 것이 전부다.
본 문서는 이클립스 개발환경을 기준으로 설명한다.

1. <a href="https://guide.rationalowl.com/library" target="_blank">JAVA 앱서버 라이브러리를 다운</a> 받는다.


2. 이클립스 프로젝트를 생성하고 다운받은 라이브러리를 libs 폴더내에 복사한다.

![이미지 이름](./img/lib_add.png)

3. 이클립스 프로젝트에 라이브러리를 추가한다.


>## 앱서버 등록(구동)
앱서버가 래셔널아울 API를 통해 단말앱과 실시간 데이터를 전달하기 위해서는 먼저 앱 서버를 등록해야 한다. 참고로 하나의 래셔널아울 고객 서비스 내에서 다수의 앱 서버 등록이 가능하다. 무료 평가판의 경우 최대 10개까지 고객 앱 서버 등록이 가능하다.

앱서버 등록 API는 앱서버를 구동시키는 역할도 한다. 따라서 앱서버가 구동시마다 앱서버 등록 API는 호출되어야 한다. 앱서버는 앱서버 등록 API 인자로 들어가는 단말 등록 이름을 기억하고 있어야 하고 앱서버 등록 API호출시 동일한 단말 등록 이름으로 호출해야 한다.

래셔널아울 관리자 콘솔은 앱서버 등록 결과에 대해 실시간 모니터링을 제공한다.


![이미지 이름](./img/reg.png)

### 앱서버 등록 요청

registerAppServer() API를 통해 앱서버를 원하는 서비스의 서버로 등록 요청한다.
주의할 점은 API호출시 전달하는 앱서버 등록이름과 등록결과 발급받은 앱서버 등록 아이디를 반드시 저장/관리해야 한다. 앱서버 등록 요청시 앱서버 등록 이름이 바뀌면 래셔널아울 서비스에서는 다른 앱서버가 등록요청하는 것으로 인식한다. registerAppServer()는 최초 앱서버 등록시 뿐 아니라 이후 서버 재가동 시 마다 해당 API를 호출해야 하는데 등록이름을 관리하지 않고 다른 이름으로 바뀌면 서버 재가동시마다 앱 서버가 추가 등록되고 새로운 단말 등록 아이디가 발급되기 때문이다. 기존 앱서버와 통신하던 단말앱에서 업스트림을 보내려 해도 앱서버 등록 아이디가 다르면 업스트림을 전달할 수 없다.


```java
AppServerManager serverMgr = AppServerManager.getInstance();
// 공용에디션일 경우
serverMgr.registerAppServer("myServiceId", "app server1", "gate.rationalowl.com", 9080);

// 전용에디션(구축형) 앱서버와 메시징 서버가 같은 사설망에서 운영시 
serverMgr.registerAppServer("myServiceId", "app server1", "192.168.0.11", 9080, true);

```

### 앱서버 등록 결과
앱서버 등록 결과는 AppServerRegisterResultListener 인터페이스의 onRegisterResult()를 통해 알 수 있다. 
앱서버는 등록 성공시 다음의 두 가지를 반드시 저장/관리해야 한다.
 1. 발급되는 앱서버 등록 아이디
    - 앱 서버를 구분하는 구분자
    - 앱 서버 API호출시 인자로 이용
    - 단말앱에서 업스트림 메시지 발신시 대상 앱서버 지정하는데 이용
        - 단말앱 개발자는 관리자 콘솔의 앱서버 현황 화면에서 앱서버 등록 아이디를 확인할 수 있다.
    - 
 2. 앱서버 등록 요청시 입력한 앱 서버 등록 이름

```java
public void onRegisterResult(int resultCode, String resultMsg, String appServerRegId, String appServerRegName) {
    output("RegisterResult:" + resultMsg + "  reg server id = " + appServerRegId + " reg server name = " + appServerRegName);

    if (resultCode == Result.RESULT_OK || resultCode == Result.RESULT_SERVER_REGNAME_ALREADY_REGISTERED) {
        // should save and manage mServerRegName and mServerRegId to the file or db.
    }
}
```


## 앱서버 등록 해제
고객 서비스 내에서 사용하지 않는 앱 서버를 등록 해제한다. 
래셔널아울 관리자 콘솔은 앱서버 등록해제 결과에 대해 실시간 모니터링을 제공한다.

### 앱서버 등록 해제 요청

unregisterAppServer() API를 등록해제 요청한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
serverMgr.unregisterAppServer("myServiceId", "app server1", "gate.rationalowl.com", 9080);
```

### 앱서버 등록 해제 결과
앱서버 등록 결과는 AppServerRegisterResultListener 인터페이스의 onUnregisterResult()를 통해 알 수 있다. 

```java
public void onUnregisterResult(int resultCode, String resultMsg) {
    output("UnregisterResult:" + resultMsg);
}

```

>## 단말그룹


>앱서버는 실시간 메시지를 주고 받기 위해 단말 앱 등록 아이디를 관리해야 한다. 즉, 단말앱도 앱스토어에서 앱을 다운받아 설치 후 최초 1회 단말 등록을 하고, 등록 결과 단말 등록 아이디를 발급받는다. 단말앱은 발급받은 단말 등록 아이디를 앱서버에게 upstream API를 통해 전달해주어야 하고 앱서버는 해당 단말 등록 아이디를 저장/관리해야 한다. 이후 단말로 다운스트림을 보낼때나 단말그룹을 관리할 때 단말 등록 아이디를 통해 관리하게 된다.


단말그룹은 그룹 메시지를 지원하기 위해서 생겨난 논리적인 단말들의 집합이다. 래셔널아울 서비스에서 제공하는 단말그룹의 특성은 다음과 같다.
  - 단말그룹은 앱서버에서 생성/단말 추가/단말 삭제/그룹 삭제가 가능하다.
  - 현재 그룹메시지는 다운스트림만 지원한다. (2018년 상반기 P2P 지원 예정)
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

registerAppServer() API를 통해 단말그룹 생성을 요청한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
ArrayList<String> devices = new ArrayList<String>();
devices.add("device registration id1");
devices.add("device registration id2");
serverMgr.createDeviceGroup("myDeviceGroup", "my first device group", devices);
```

### 단말그룹 생성 결과
단말그룹 생성 결과는 DeviceGroupListener 인터페이스의 onDeviceGroupCreateResult()를 통해 알 수 있다. 
앱서버는 등록 성공시 발급되는 단말그룹아이디를 반드시 저장/관리해야 한다. 이후 그룹메시지 발신시 그룹아이디를 통해 그룹메시지를 발신한다.

```java
public void onDeviceGroupCreateResult(int resultCode, String resultMsg, String deviceGrpId, String deviceGrpName, int deviceSize, String desc, ArrayList<String> failedDevices, String requestId) {
       
    System.out.println("onUpstreamMsgReceived deviceGrpId " + deviceGrpId + "msg = " + resultMsg);
    
    if(resultCode == Result.RESULT_OK) {     
        // should save deviceGrpId and manager it.           
    }
}
```

## 단말그룹 내 단말 추가
생성된 단말 그룹에 단말을 추가한다.
한번에 추가할 수 있는 최대 단말 수는 2000개이다.
단말 그룹 추가 API를 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.

### 단말그룹 내 단말 추가 요청

addDeviceGroup() API를 통해 단말그룹에 단말 추가 요청한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();           
ArrayList<String> devices = new ArrayList<String>();
devices.add("device registration id3");
devices.add("device registration id4");
serverMgr.addDeviceGroup("deviceGroupId1", devices);
```

### 단말그룹 내 단말 추가 결과
단말그룹 내 단말 추가 결과는 DeviceGroupListener 인터페이스의 onDeviceGroupAddResult()를 통해 알 수 있다. 

```java
public void onDeviceGroupAddResult(int resultCode, String resultMsg, String deviceGrpId, int totalDeviceSize, int addedDeviceSize, ArrayList<String> failedDevices, String requestId) {
    System.out.println("onDeviceGroupAddResult deviceGrpId " + deviceGrpId + "msg = " + resultMsg);
    
    if(resultCode == Result.RESULT_OK) {
        // devices has added to device group successfuly.
        // do something you want here
    }
}
```

## 단말그룹 내 단말 제거
생성된 단말 그룹에서 단말을 제거한다.
한번에 제거할 수 있는 최대 단말 수는 2000개이다.
단말 그룹 관리 API를 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.

### 단말그룹 내 단말 제거 요청

subtractDeviceGroup() API를 통해 단말그룹에서 단말 제거 요청한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();           
ArrayList<String> devices = new ArrayList<String>();
devices.add("device registration id3");
devices.add("device registration id4");
serverMgr.subtractDeviceGroup("deviceGroupId1", devices);
```

### 단말그룹 내 단말 제거 결과
단말그룹 내 단말 제거 결과는 DeviceGroupListener 인터페이스의 onDeviceGroupSubtractResult()를 통해 알 수 있다. 

```java
 public void onDeviceGroupSubtractResult(int resultCode, String resultMsg, String deviceGrpId, int totalDeviceSize, int subtractDeviceSize, ArrayList<String> failedDevices, String requestId) {
    System.out.println("onDeviceGroupSubtractResult deviceGrpId " + deviceGrpId + "msg = " + resultMsg);
    
    if(resultCode == Result.RESULT_OK) {
       // devices has removed from device group successfuly.
       // do something you want here
    }
}
```


## 단말그룹 삭제
단말그룹을 삭제한다.
단말그룹을 삭제하더라도 단말그룹 내 단말이 등록해제되는 것은 아니다.
단말 그룹 관리 API를 연속적으로 호출할 경우 이전 호출 결과 콜백 이후 다음 호출을 진행해야 한다.

### 단말그룹 삭제 요청

deleteDeviceGroup() API를 통해 단말그룹 삭제 요청한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();           
serverMgr.deleteDeviceGroup("deviceGroupId1");
```

### 단말그룹 삭제 결과
단말그룹 내 단말 추가 결과는 DeviceGroupListener 인터페이스의 onDeviceGroupDeleteResult()를 통해 알 수 있다. 

```java
public void onDeviceGroupDeleteResult(int resultCode, String resultMsg, String deviceGrpId, String requestId) {
    System.out.println("onDeviceGroupDeleteResult deviceGrpId " + deviceGrpId + "msg = " + resultMsg);
    
    if(resultCode == Result.RESULT_OK) {
       // devices group has removed successfully.
       // do something you want here
    }      
}
```


>## 다운스트림 메시지

앱서버에서 단말앱으로의 다운스트림 메시지는 다운스트림 API 파라미터에 따라 실시간 전용, 푸시 알림 지원, 메시지 큐잉 지원의 옵션으로 사용된다. 그리고 래셔널아울 관리자 콘솔에서 제공하는 데이터 전달 현황 실시간 모니터링은 고객 서비스 개발시에는 개발의 용이함을 제공하고 서비스 운영시에는 서비스 대응력을 높이고 예측 가능성을 향상시킨다.

래셔널아울에서 지원하는 다운스트림 메시지의 특성은 다음과 같다.

  - 단말앱이 실행시에는 평균 0.2초에 데이터 전달을 제공한다.
  - 단말앱이 미실행시에는 푸시 알림을 지원한다.
  - 다운스트림 API에서 단말앱이 네트워크에 연결되지 않을 경우 전달 데이터외에 알림 타이틀과 알림 문자를 별도로 지정할 수 있다.  
  - 단말앱이 네트워크에 연결되지 않을 경우 큐잉 후 단말이 네트워크 접속시 전달하는 큐잉을 지원한다.
  - 기본 큐잉 기간은 3일이고 엔터프라이즈 에디션에서는 최대 30일까지 설정 가능하다.
  - 푸시알림 이용, 큐잉 기능 이용 여부는 앱서버 라이브러리에서 제공하는 다운스트림 API 호출시 옵션으로 지정한다.
  - 래셔널아울 콘솔은 데이터 전달 현황에 대해 실시간 모니터링을 제공한다.


### 메시지 전송시간
래셔널아울 서비스는 평균 0.2초의 메시지 전송속도를 제공한다. 이는 다운스트림 메시지뿐 아니라 업스트림과 P2P 메시지 모두에 해당된다. 메시지 전송속도는 발신주체에서 메시지를 발신하여 수신주체가 메시지를 수신할 때까지 소요된 시간을 의미한다.  

![이미지 이름](./img/delivery_time.png)

## 멀티캐스트
  - 한대 이상의 단말앱에 메시지를 발신한다.  
  - 한번에 보낼 수 있는 최대 대상 단말 수는 2000대이다.
  
### 멀티캐스트 메시지 발신
sendMulticastMsg() API를 통해 멀티캐스트 메시지를 발신한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
String data = "multicast message test";
ArrayList<String> devices = new ArrayList<String>();
devices.add("device registration id1");
devices.add("device registration id2");
boolean supportMsgQ = true;
String notiTitle = null;
String notiBody = null;
serverMgr.sendMulticastMsg(data, regIDs, supportMsgQ, notiTitle, notiBody);
// same as serverMgr.sendMulticastMsg(data, regIDs);
```

### 멀티캐스트 메시지 발신 결과
메시지 발신 결과는 MessageListener 인터페이스의 onSendMulticastMsgResult()를 통해 알 수 있다.
해당 콜백은 메시지 발신 성공 여부에 대해서만 알려준다. 실제 앱서버 개발시 본 콜백은 개발시에만 사용하기를 권고한다. 이보다는 래셔널아울 관리자 콘솔의 메시지 현황 모니터링은 개발 단계 뿐 아니라 서비스 운영단계에서 훨씬 유용하다.

```java
public void onSendMulticastMsgResult(int resultCode, String resultMsg, String requestId) {
    System.out.println("onSendMulticastMsgResult resultCode " + resultCode);
    
    if(resultCode == Result.RESULT_OK) {
       // multicast message has sent successfully
    }      
}
```


## 브로드캐스트
  - 고객 서비스내에 등록된 모든 단말앱들에게 메시지를 발신한다.
  - 브로드캐스트 메시지는 실시간 데이터 전달의 용도로 이용되어서는 안된다.
  - 브로드캐스트 메시지는 전체 서비스내 단말앱에 푸시 알림 용도로만 사용해야 한다.
  
### 브로드캐스트 메시지 발신
sendBroadcastMsg() API를 통해 브로드캐스트 메시지를 발신한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
String data = "broadcast message test";
boolean supportMsgQ = true;
String notiTitle = null;
String notiBody = null;
serverMgr.sendBroadcastMsg(data, supportMsgQ, notiTitle, notiBody);
// same as serverMgr.sendBroadcastMsg(data);
```

### 브로드캐스트 메시지 발신 결과
메시지 발신 결과는 MessageListener 인터페이스의 onSendBroadcastMsgResult()를 통해 알 수 있다.
해당 콜백은 메시지 발신 성공 여부에 대해서만 알려준다. 실제 앱서버 개발시 본 콜백은 개발시에만 사용하기를 권고한다. 이보다는 래셔널아울 관리자 콘솔의 메시지 현황 모니터링은 개발 단계 뿐 아니라 서비스 운영단계에서 훨씬 유용하다.

```java
public void onSendBroadcastMsgResult(int resultCode, String resultMsg, String requestId) {
    System.out.println("onSendBroadcastMsgResult resultCode " + resultCode);
    
    if(resultCode == Result.RESULT_OK) {
       // broadcast message has sent successfully
    }      
}
```


## 그룹메시지
그룹메시지는 단말그룹에게 메시지를 발신하는 방식으로 기능적으로 전통적인 통신방식인 멀티캐스트와 동일하다.
그룹 메시지 방식은 효율적이고 확장성이 높은 pub/sub 메시지 모델로 구현되었으며 동일한 대상 단말수일 경우 그룹 메시지 방식이 약 20% 전송속도가 빠르다. 따라서 1회성 또는 단발성 통신을 제외하고는  멀티캐스트 API보다는 그룹메시지 API의 사용을 권고한다.
  
### 그룹메시지 발신
sendBroadcastMsg() API를 통해 브로드캐스트 메시지를 발신한다.

```java
String data = "group message test";
String grpId = grp.mDeviceGrpId;       
boolean supportMsgQ = true;
String notiTitle = null;
String notiBody = null;
serverMgr.sendGroupMsg(data, grpId, supportMsgQ, notiTitle, notiBody);
// same as serverMgr.sendGroupMsg(data, grpId);
```

### 그룹메시지 발신 결과
메시지 발신 결과는 MessageListener 인터페이스의 onSendGroupMsgResult()를 통해 알 수 있다.
해당 콜백은 메시지 발신 성공 여부에 대해서만 알려준다. 실제 앱서버 개발시 본 콜백은 개발시에만 사용하기를 권고한다. 이보다는 래셔널아울 관리자 콘솔의 메시지 현황 모니터링이 개발 단계 뿐 아니라 서비스 운영단계에서 훨씬 유용하다.

```java
public void onSendGroupMsgResult(int resultCode, String resultMsg, String requestId) {
    System.out.println("onSendGroupMsgResult resultCode " + resultCode);
    
    if(resultCode == Result.RESULT_OK) {
       // group message has sent successfully
    }      
}
```

>## 업스트림 메시지 수신

앱서버가 업스트림 메시지를 수신하면 onUpstreamMsgReceived() 콜백이 호출된다.


```java
class SimpleMessageListener implements MessageListener {

    @Override
    public void onUpstreamMsgReceived(String sender, long sendTime, String msg) {
        output("onUpstreamMsgReceived sender device reg id : " + sender + "send time:" + sendTime + " msg : " + msg);
    }
}
```


>## 커스텀푸시

이미지 푸시와 팝업 푸시 등 앱 성격에 맞게 푸시알림의 형태를 임의로 표현할 수 있다. 리치 노티피케이션(Rich Notification)이라고도 회자되는데 커스텀푸시(Customizable Push)라는 용어가 가장 적절할 것 같다. 래셔널아울 서비스에서는 이를 커스텀 푸시라는 용어로 통일하여 사용한다. 커스텀푸시는 기존 리치 노티피케이션에 실시간 트래킹과 모니터링 기능을 추가한 개념이다.

커스텀 푸시의 전반적인 개념은 아래 서비스 블로그 문서를 참조하는 것이 효과적이다.

- [커스텀푸시 개념](https://rationalowl.tistory.com/20) 

## 커스텀푸시 멀티캐스트
  - 한대 이상의 단말앱에 메시지를 발신한다.  
  - 한번에 보낼 수 있는 최대 대상 단말 수는 2000대이다.
  
### 커스텀푸시 멀티캐스트 메시지 발신
sendMulticastCustomPush() API를 통해 멀티캐스트 메시지를 발신한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
String data = "multicast message test";
ArrayList<String> devices = new ArrayList<String>();
devices.add("device registration id1");
devices.add("device registration id2");
Map<String, Object> data = new HashMap<String, Object>();
data.put("myNotiTitle", "커스텀 알람 타이틀");
data.put("myNotiBody", "이미지url로 나만의 알림을 표현하세요!");
data.put("imageUrl", "http://your.image.com/image1.jpg");
data.put("soundUrl", "http://your.sound.com/sound1.wav");           
serverMgr.sendMulticastCustomPush(data, regIDs);
```

### 커스텀푸시 멀티캐스트 메시지 발신 결과
메시지 발신 결과는 MessageListener 인터페이스의 onSendMulticastCustomPushResult()를 통해 알 수 있다.
해당 콜백은 메시지 발신 성공 여부에 대해서만 알려준다. 

```java
public void onSendMulticastCustomPushResult(int resultCode, String resultMsg, String msgId, String requestId) {
    output("onSendMulticastMsgResult:" + resultCode + "msgId = " + msgId);

    if(resultCode == Result.RESULT_OK) {
       // multicast message has sent successfully
    }    
}
```


## 커스텀푸시 브로드캐스트
  - 고객 서비스내에 등록된 모든 단말앱들에게 메시지를 발신한다.
  
### 커스텀푸시 브로드캐스트 메시지 발신
sendBroadcastCustomPush() API를 통해 브로드캐스트 메시지를 발신한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
Map<String, Object> data = new HashMap<String, Object>();
data.put("key1",  "dddd");
data.put("key2", 2);
data.put("key3", "value3");           
serverMgr.sendBroadcastCustomPush(data);
```

### 커스텀푸시 브로드캐스트 메시지 발신 결과
메시지 발신 결과는 MessageListener 인터페이스의 onSendBroadcastMsgResult()를 통해 알 수 있다.

```java
public void onSendBroadcastCustomPushResultonSendBroadcastCustomPushResult(int resultCode, String resultMsg, String msgId, String requestId) {
    output("onSendBroadcastCustomPushResult:" + resultCode + "msgId = " + msgId);
    
    if(resultCode == Result.RESULT_OK) {
       // broadcast message has sent successfully
    }      
}
```


## 커스텀푸시 그룹메시지
그룹메시지는 단말그룹에게 메시지를 발신하는 방식으로 기능적으로 전통적인 통신방식인 멀티캐스트와 동일하다.
그룹 메시지 방식은 효율적이고 확장성이 높은 pub/sub 메시지 모델로 구현되었으며 동일한 대상 단말수일 경우 그룹 메시지 방식이 약 20% 전송속도가 빠르다. 따라서 1회성 또는 단발성 통신을 제외하고는  멀티캐스트 API보다는 그룹메시지 API의 사용을 권고한다.
  
### 커스텀푸시 그룹메시지 발신
sendBroadcastMsg() API를 통해 브로드캐스트 메시지를 발신한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
String grpId = "deviceGroupId1";
Map<String, Object> data = new HashMap<String, Object>();
data.put("key1", "value1");
data.put("key2", 2);
data.put("key3", "value3");   
serverMgr.sendGroupCustomPush(data, grpId);
```

### 커스텀푸시 그룹메시지 발신 결과
메시지 발신 결과는 MessageListener 인터페이스의 onSendGroupCustomPushResult()를 통해 알 수 있다.

```java
public void onSendGroupCustomPushResult(int resultCode, String resultMsg, String msgId, String requestId) {
    output("onSendGroupCustomPushResult:" + resultCode + "msgId = " + msgId);
    
    if(resultCode == Result.RESULT_OK) {
       // group message has sent successfully
    }      
}
```


## 커스텀푸시 재발신
앞서 위의 세 커스텀 푸시 발신 API로 발신 후 알림 미전달 혹은 알림수신 미확인 대상 사용자 폰에 재발신한다.
  
### 커스텀푸시 그룹메시지 발신
retryCustomPush() API를 통해 재발신한다.

```java
AppServerManager serverMgr = AppServerManager.getInstance();
String msgId = "your msg id";
int retryCondition = 1; // 1: 알림미전달 단말 대상 재 발신, 2: 수신미확인 단말 대상 재 발신     
serverMgr.retryCustomPush(msgId, 1);  
```


>## 메시지 트래킹


>메시지 트래킹은 발신한 메시지의 실시간 전달상태를 알려준다.  
메시지 트래킹 API의 호출을 통해 결과를 API콜백에서 확인한다.




## 메시지 트래킹(메시지 통계 정보)

  한번에 다수의 메시지 전달통계를 실시간 확인한다. 
  - 커스텀푸시 및 실시간 메시지 모두 지원한다.
  - 한번에 최대 100개 메시지 아이디 지정
  - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.   
  
### 메시지 트래킹(메시지 통계 정보) API
 

```java
AppServerManager serverMgr = AppServerManager.getInstance();   
ArrayList<String> msgIds = new ArrayList<String>();
msgIds.add("msgId1");
msgIds.add("msgId2");
...
msgIds.add("msgId100");
serverMgr.msgStatsTracking(msgIds);
// same as serverMgr.sendBroadcastMsg(data);
output("msg stats tracking request msgId list :" + msgIds);
```

### 메시지 트래킹(메시지 통계 정보) API 결과
메시지 발신 결과는 MsgTrackingListener 인터페이스의 onMsgStatsTrackingResult()를 통해 알 수 있다.

```java
class SimpleTrackingListener implements MsgTrackingListener {

    @Override
    /**
     * AppServerManager의  msgStatsTracking()API 호출결과 콜백이다.
     * 
     * @param resultCode
     *            Result클래스에 정의된 결과값 상수
     * @param resultMsg
     *            resultCode의 값에 대한 의미
     * @param msgStats
     *            메시지 전달 현황 통계 map의 배열
     *            [
     *              { 
     *                "mId":"msgid1",   // msgId(메시지 아이디)
     *                "t":1200,         // total(메시지 발신 대상 총 단말앱 수)
     *                "q":100,          // 미전달 큐잉 단말앱 수
     *                "n":900,          // notiDeliver (알림 전달 받은 단말앱 수)
     *                "d":200,          // deliver 사용자 수신 확인(데이터 전달) 단말앱 수
     *                "e":0             // expire 미전달 기간 만료된 대상 단말앱 수 
     *              }, 
     *              { 
     *                "mId":"msgid2",   
     *                "t":1200,         
     *                "q":100,          
     *                "n":900,          
     *                "d":200,          
     *                "e":0              
     *              },
     *              ...
     *            ]
     * @param requestId
     *            본 콜백 결과를 야기한 createDeviceGroup() API 반환값
     */
    public void onMsgStatsTrackingResult(int resultCode, String resultMsg, ArrayList<Map<String, Object>> msgStats, String requestId) {
        output("onMsgStatsTrackingResult:" + resultCode + "msg = " + resultMsg  + "msgStats = " + msgStats );
    }
}
```



## 메시지 상태 트래킹(메시지 상태별 단말앱 목록)

  하나의 커스텀푸시 메시지 상태별 단말앱 목록을 실시간 트래킹한다.
 - 알림 미전달 단말앱 목록만 트래킹하여 알림 미전달 단말앱 대상 유료 메시지 발신시 유용하다.   
 - 멀티캐스트, 그룹메시지, 브로드캐스트로 동시에 1000단말앱이상 발신한 메시지의 트래킹에 효과적이다.
 - 100만 단말앱에 동시에 커스텀 푸시 발신시에도 1분대에 전체 발신 및 전달 및 수신확인 여부 실시간 트래킹이 가능하다.
 - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.   
  
### 메시지 상태 트래킹(메시지 상태별 단말앱 목록) API
 
```java
    /**
     * 메시지 상태별 단말앱 목록을 실시간 확인한다. 가장 일반적인 사용예는 알림 미전달 대상 단말앱 목록에 유료 메시지 발신하고자 할
     * 경우이다.
     * 
     * @param msgId
     *            트래킹하고자 하는 메시지 아이디
     * @param msgStateFlag
     *            : 아래 값들의 bit or 값 0: all 1: 큐잉된 단말앱 목록을 fetch한다. 2: 수신확인한 단말앱
     *            목록을 fetch한다. 4: 메시지 미전달 기한만료된 단말앱 목록을 fetch한다. 8: 알림전달한 단말앱
     *            목록을 fetch한다.
     * @param deviceTypeFlag
     *            : 아래 값들의 bit or 값 0: all 1: 안드로이드 단말앱 목록을 fetch한다. 2: IOS 단말앱
     *            목록을 fetch한다.
     * @param startIndex
     *            전체 단말앱 중 fetch 시작 index
     * @param fetchSize
     *            전체 단말앱 목록 startIndex부터 fetch할 단말앱 목록 수
     * @return request id
     */
    public String msgDevicesTracking(String msgId, int msgStateFlag, int deviceTypeFlag, int startIndex, int fetchSize)
```

API 호출 예

```java
AppServerManager serverMgr = AppServerManager.getInstance(); 
serverMgr.msgDevicesTracking(msgId, msgState, deviceType, startIdx, fetchSize);
```

### 메시지 상태 트래킹(메시지 상태별 단말앱 목록) API 결과
메시지 발신 결과는 MsgTrackingListener 인터페이스의 onMsgDevicesTrackingResult()를 통해 알 수 있다.

```java
class SimpleTrackingListener implements MsgTrackingListener {

    @Override
    /**
     * AppServerManager의 msgDetailTracking()API 호출결과 콜백이다.
     * 
     * @param resultCode
     *            Result클래스에 정의된 결과값 상수
     * @param resultMsg
     *            resultCode의 값에 대한 의미
     * @param msgId
     *            상세 메시지 전달 내역 대상 메시지 아이디
     * @param startIndex
     *            메시지 전달 대상 전체 단말앱 중 fetch 한 첫 번째 인덱스
     * @param totalDeviceSize
     *            메시지 전달 대상 전체 단말앱
     * @param fetchDeviceSize
     *            totalDeviceSize 중 startIndex부터 fetchDeviceSize 만큼의 단말앱 정보를 msgDetail에 fetch 해 온다.
     * @param deviceList
     *            fetch 해 온 메시지 전달 대상 단말 등록 아이디 목록            
     *                      
     * @param requestId
     *            본 콜백 결과를 야기한 addDeviceGroup() API 반환값
     */
    public void onMsgDevicesTrackingResult(int resultCode, String resultMsg, String msgId, int startIndex, int totalDeviceSize, int fetchDeviceSize, ArrayList<String> deviceList, String requestId) {
        output("onSendGroupCustomPushResult:" + resultCode + "msg = " + resultMsg + " msgId =" + msgId + " startIndex=" + startIndex + " totalDeviceSize=" + totalDeviceSize 
                + " fetchDeviceSize = " + fetchDeviceSize + " deviceList = " + deviceList);
    }
}
```


## 메시지 상태 트래킹(메시지 상태별 단말앱 상세정보)

  하나의 커스텀푸시 메시지 상태별 단말앱 상세정보를 실시간 트래킹한다.
   - 멀티캐스트, 그룹메시지, 브로드캐스트로 동시에 1000단말앱이상 발신한 메시지의 트래킹에 효과적이다.
   - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.   
  
### 메시지 상태 트래킹(메시지 상태별 단말앱 상세정보) API
 
 ```java
    /**
     * 메시지 상태별 단말앱 상세정보를 실시간 확인한다.
     * 
     * @param msgId
     *            트래킹하고자 하는 메시지 아이디
     * @param msgStateFlag
     *            : 아래 값들의 bit or 값 0: all 1: 큐잉된 단말앱 목록을 fetch한다. 2: 수신확인한 단말앱
     *            목록을 fetch한다. 4: 메시지 미전달 기한만료된 단말앱 목록을 fetch한다. 8: 알림전달한 단말앱
     *            목록을 fetch한다.
     * @param deviceTypeFlag
     *            : 아래 값들의 bit or 값 0: all 1: 안드로이드 단말앱 목록을 fetch한다. 2: IOS 단말앱
     *            목록을 fetch한다.
     * @param startIndex
     *            전체 단말앱 중 fetch 시작 index
     * @param fetchSize
     *            전체 단말앱 목록 startIndex부터 fetch할 단말앱 목록 수
     * @return request id
     */
    public String msgDeviceDetailTracking(String msgId, int msgStateFlag, int deviceTypeFlag, int startIndex, int fetchSize)
```

API 호출 예

```java
AppServerManager serverMgr = AppServerManager.getInstance(); 
serverMgr.msgDeviceDetailTracking(msgId, msgState, deviceType, startIdx, fetchSize);
```

### 메시지 상태 트래킹(메시지 상태별 단말앱 상세정보) API 결과
메시지 발신 결과는 MsgTrackingListener 인터페이스의 onMsgDeviceDetailTrackingResult()를 통해 알 수 있다.

```java
class SimpleTrackingListener implements MsgTrackingListener {

    @Override
    /**
     * AppServerManager의 msgDetailTracking()API 호출결과 콜백이다.
     * 
     * @param resultCode
     *            Result클래스에 정의된 결과값 상수
     * @param resultMsg
     *            resultCode의 값에 대한 의미
     * @param msgId
     *            상세 메시지 전달 내역 대상 메시지 아이디
     * @param startIndex
     *            메시지 전달 대상 전체 단말앱 중 fetch 한 첫 번째 인덱스
     * @param totalDeviceSize
     *            메시지 전달 대상 전체 단말앱
     * @param fetchDeviceSize
     *            totalDeviceSize 중 startIndex부터 fetchDeviceSize 만큼의 단말앱 정보를 msgDetail에 fetch 해 온다.
     * @param msgDetail
     *            fetch 해 온 메시지 전달 대상 단말 정보 목록
     *            메시지 전달 대상 단말 map 의 배열 
     *            [
     *              {
     *                "drId":"12122123",           // deviceRegId 단말 등록 아이디
     *                "drn": "john's i phone",     // deviceRegName 단말 등록 이름
     *                "os": 2,                     // 단말 OS (1: android, 2: ios)
     *                "ms":1,                      // msgState 단말의 메시지 전달 상태 (1: 미전달 큐잉, 2: 수신확인(데이터 전달), 4(미전달 기간만료), 8(알림 전달)
     *                "dt":11111,                  // deliverTime(메시지 수신확인 시간: 밀리세컨드 단위): optional field로 msgState가 2인 경우 세팅된다.                                 
     *                "nt":22222                   // notification delivery time(알림 전달 시간: 밀리세컨드 단위) optional field로  msgState가 8인 경우 세팅된다.
     *              }
     *            ]
     *                      
     * @param requestId
     *            본 콜백 결과를 야기한 addDeviceGroup() API 반환값
     */
    public void onMsgDeviceDetailTrackingResult(int resultCode, String resultMsg, String msgId, int startIndex, int totalDeviceSize, int fetchDeviceSize, ArrayList<Map<String, Object>> msgDetail, String requestId) {
        // TODO Auto-generated method stub
        output("onSendGroupCustomPushResult:" + resultCode + "msg = " + resultMsg + " msgId =" + msgId + " startIndex=" + startIndex + " totalDeviceSize=" + totalDeviceSize 
                + " fetchDeviceSize = " + fetchDeviceSize + " msgDetail = " + msgDetail);
    }
}
```


## 배치 메시지 트래킹(메시지 상태별 단말앱 목록)

  한번에 다수의 메시지를 트래킹시 사용한다.
   - 커스텀푸시 메시지 상태별 단말앱 목록을 실시간 트래킹한다.
   - 개별로 전달하는 개인화 메시지나 대상 단말앱이 1000미만인 메시지인 경우 동시에 여러 메시지에 대한 트래킹시 이용한다.
   - 한번에 최대 100메시지 트래킹 지원
   - 전체 메시지의 대상 단말앱 수가 1000미만이어야 한다.
   - 알림 미전달 단말앱 목록만 트래킹하여 알림 미전달 단말앱 대상 유료 메시지 발신시 유용하다.
   - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.     
  
### 배치 메시지 트래킹(메시지 상태별 단말앱 목록) API
 
```java
    /**
     * 다수의 메시지 상태별 단말앱 목록을 실시간 확인한다. 대상 단말이 1~2대인 개인화된 메시지의 트래킹 정보를 고속으로 처리하고자
     * 할때 사용한다. 전체 메시지의 대상 단말앱 수가 1000개를 넘어서는 안된다. 대상 단말앱수가 과다할 경우 배치 API 대신 일반
     * msgDevicesTracking API를 이용해야 한다. 가장 일반적인 사용예는 알림 미전달 대상 단말앱 목록에 유료 메시지
     * 발신하고자 할 경우이다.
     * 
     * 
     * @param msgIds
     *            메시지 상태별 단말아이디 목록을 알고자 하는 메시지 아이디 목록 메시지 아이디 목록은 최대 100개까지
     *            지원한다.
     * 
     * @param msgStateFlag
     *            : 아래 값들의 bit or 값 0: all 1: 큐잉된 단말앱 목록을 fetch한다. 2: 수신확인한 단말앱
     *            목록을 fetch한다. 4: 메시지 미전달 기한만료된 단말앱 목록을 fetch한다. 8: 알림전달한 단말앱
     *            목록을 fetch한다.
     * @return request id
     */
    public String batchMsgDevicesTracking(ArrayList<String> msgIds, int msgStateFlag)
```

API 호출 예
```java
AppServerManager serverMgr = AppServerManager.getInstance();   
serverMgr.batchMsgDevicesTracking(msgIds, msgStateFlag);
```

### 배치 메시지 트래킹(메시지 상태별 단말앱 목록) API 결과
메시지 발신 결과는 MsgTrackingListener 인터페이스의 onBatchMsgDeviceDetailTrackingResult()를 통해 알 수 있다.

```java
class SimpleTrackingListener implements MsgTrackingListener {

    @Override
    /**
     * AppServerManager의 batchMsgDevicesTracking()API 호출결과 콜백이다.
     * 
     * @param resultCode
     *            Result클래스에 정의된 결과값 상수
     * @param resultMsg
     *            resultCode의 값에 대한 의미
     * @param msgs
     *            메시지 Id와 단말등록아이디 배열
     *            [
     *              { 
     *                "mId":"msgid1",   // msgId(메시지 아이디)
     *                "dl":["111111","2222222"]     // deviceId list 
     *              },
     *              ....
     *              { 
     *                "mId":"msgid100",
     *                "dl":["111111","2222222"] 
     *              }
     *            ]
     * @param requestId
     *            본 콜백 결과를 야기한 createDeviceGroup() API 반환값
     */
    public void onBatchMsgDevicesTrackingResult(int resultCode, String resultMsg, ArrayList<Map<String, Object>> msgs, String requestId) {
        // TODO Auto-generated method stub
        output("onBatchMsgDevicesTrackingResult:" + resultCode + "msg = " + resultMsg  + "msgs = " + msgs );
    }
}
```

## 배치 메시지 트래킹(메시지 상태별 단말앱 상세정보)

 한번에 다수의 메시지를 트래킹시 사용한다.
   - 커스텀푸시 메시지 상태별 단말앱 상세정보를 실시간 트래킹한다.
   - 개별로 전달하는 개인화 메시지나 대상 단말앱이 1000미만인 메시지인 경우 동시에 여러 메시지에 대한 트래킹시 이용한다.
   - 한번에 최대 100메시지 트래킹 지원
   - 전체 메시지의 대상 단말앱 수가 1000미만이어야 한다.
   - 메시지 큐잉기간(기본 3일)내 발신한 메시지에 대한 트래킹 지원.
  
### 배치 메시지 트래킹(메시지 상태별 단말앱 상세정보) API
 ```java
    /**
     * 다수의 메시지 상태별 단말앱 상세 정보를 실시간 확인한다. 대상 단말이 1~2대인 개인화된 메시지의 트래킹 정보를 고속으로
     * 처리하고자 할때 사용한다. 전체 메시지의 대상 단말앱 수가 1000개를 넘어서는 안된다. 대상 단말앱수가 과다할 경우 배치 API
     * 대신 일반 msgDeviceDetailTracking API를 이용해야 한다.
     * 
     * 
     * @param msgIds
     *            메시지 상태별 단말아이디 목록을 알고자 하는 메시지 아이디 목록 메시지 아이디 목록은 최대 100개까지
     *            지원한다.
     * 
     * @param msgStateFlag
     *            : 아래 값들의 bit or 값 0: all 1: 큐잉된 단말앱 목록을 fetch한다. 2: 수신확인한 단말앱
     *            목록을 fetch한다. 4: 메시지 미전달 기한만료된 단말앱 목록을 fetch한다. 8: 알림전달한 단말앱
     *            목록을 fetch한다.
     * @return request id
     */
    public String batchMsgDeviceDetailTracking(ArrayList<String> msgIds, int msgStateFlag)
```

API 호출 예

```java
AppServerManager serverMgr = AppServerManager.getInstance();   
serverMgr.batchMsgDeviceDetailTracking(msgIds, msgStateFlag);
```

### 배치 메시지 트래킹(메시지 상태별 단말앱 상세정보) API 결과
메시지 발신 결과는 MsgTrackingListener 인터페이스의 onBatchMsgDeviceDetailTrackingResult()를 통해 알 수 있다.

```java
class SimpleTrackingListener implements MsgTrackingListener {

    @Override
    /**
     * AppServerManager의  batchMsgDeviceDetailTracking()API 호출결과 콜백이다.
     * 
     * @param resultCode
     *            Result클래스에 정의된 결과값 상수
     * @param resultMsg
     *            resultCode의 값에 대한 의미
     * @param msgs
     *            메시지 Id와 단말정보 배열
     *            [
     *              { 
     *                "mId":"msgid1",   // msgId(메시지 아이디)
     *                "dl":[            // device list : 단말 상세 정보 목록
     *                   {
     *                     "drId":"12122123",           // deviceRegId 단말 등록 아이디
     *                     "drn": "john's i phone",     // deviceRegName 단말 등록 이름
     *                     "os": 2,                     // 단말 OS (1: android, 2: ios)
     *                     "ms":1,                      // msgState 단말의 메시지 전달 상태 (1: 미전달 큐잉, 2: 수신확인(데이터 전달), 4(미전달 기간만료), 8(알림 전달)
     *                     "dt":11111,                  // deliverTime(메시지 수신확인 시간: 밀리세컨드 단위): optional field로 msgState가 2인 경우 세팅된다.                                 
     *                     "nt":22222                   // notification delivery time(알림 전달 시간: 밀리세컨드 단위) optional field로  msgState가 8인 경우 세팅된다.
     *                    },
     *                    ....
     *                 ]      
     *              },
     *              ....
     *              { 
     *                "mId":"msgid100",
     *                "dl":[...] 
     *              }
     *            ]
     * @param requestId
     *            본 콜백 결과를 야기한 createDeviceGroup() API 반환값
     */
    public void onBatchMsgDeviceDetailTrackingResult(int resultCode, String resultMsg, ArrayList<Map<String, Object>> msgs, String requestId) {
        // TODO Auto-generated method stub
        output("onBatchMsgDeviceDetailTrackingResult:" + resultCode + "msg = " + resultMsg  + "msgs = " + msgs );
    }
}
```


>## 메시지 트래킹 자동콜백 


>메시지 트래킹 API를 호출하면 결과 콜백에서 메시지 전달 상태를 알려준다. 즉, 메시지 전달 상태를 알기 위해서는 매번 메시지 트래킹 API를 호출해야 한다. 메시지 트래킹 자동콜백 설정을 하면 일일히 트래킹 API를 호출하지 않아도 메시지 전달 현황을 실시간 콜백으로 알려준다.


### 메시지 트래킹 자동콜백 유의사항

- 메시지 트래킹 API는 REST API도 지원하는데 반해 메시지 트래킹 자동콜백은 앱서버 라이브러리에서만 제공한다.
- 메시지 트래킹 API는 실시간 메시지와 커스텀 푸시 모두 지원하는데 반해 메시지 트래킹 자동콜백은 커스텀 푸시만 지원한다.
- 메시지 트래킹 API는 퍼블릭푸시 서버에서 응답하는 상태값을 제공하지 않는데 반해 메시지 트래킹 자동콜백은 제공한다.
- 메시지 트래킹 API는 단말앱이 삭제된 단말 대상 발신한 메시지에 대해서는 빈 데이터를 제공하는데 반해 메시지 트래킹 자동콜백은 앱삭제 상태값을 별도로 제공한다.

### 메시지 트래킹 자동콜백 모드 활성화

```java
AppServerManager serverMgr = AppServerManager.getInstance();
serverMgr.setAutoMsgTracking(true);
```

- setAutoMsgTracking(true) 를 호출하면 메시지 트래킹 자동콜백 모드가 활성화된다.
- setAutoMsgTracking(false)) 를 호출하면 메시지 트래킹 자동콜백 모드가 비활성화된다.

### 메시지 트래킹 자동콜백 설정

```java
AppServerManager serverMgr = AppServerManager.getInstance();
serverMgr.setAutoMsgTrackingListener(new SimpleAutoMsgTrackingListener());
```

- setAutoMsgTrackingListener()로 메시지 트래킹 자동콜백 지정
- 이후 커스텀푸시 발신후 메시지 전달 상태가 변경시 onAutoMsgTracked() 콜백으로 자동 호출된다.
- onAutoMsgTracked(ArrayList<Map<String, Object>> msgs) 콜백의 msgs는 json 포맷의 메시지 상태 변경 목록이 담겨있다.
- 하나의 메시지 상태 포맷은 {"mId":"message id 1", "dIds":[{"dId":"deviceId1", "ps":11, "s":2}]} 포맷이다.
- 하나의 메시지 상태 포맷의 "mId"는 메시지 아이디로 푸시 발신후 발신 결과 콜백에서 전달받은 메시지 아이디이다.
- 하나의 메시지 상태 포맷의 "dIds"는 메시지 발신 대상 단말 중 메시지 상태가 변경된 단말들의 메시지 상태 목록이다.
- "s"는 래셔널아울에서 트래킹한 메시지 전달상태
- "ps"는 퍼블릭 푸시 서버로부터 전달받은 상태
- 하나의 메시지 상태 포맷의 상세설명은 아래 소스상의 코맨트를 참조하기 바란다.


```java

class SimpleAutoMsgTrackingListener implements AutoMsgTrackingListener {

    
    /**
     * 퍼블릭 푸시 서버로부터 상태정보("ps")를 받거나 래셔널아울 서버가 메시지 상태 변경을 감지한 메시지 목록을 알려준다.
     * 
     * @param msgs
     *            발신한 메시지의 대상 단말별 메시지 상태 변화가 생긴 목록들을 배열형태로 전달 받는다. 배열에는 다수의 메시지 전달 상태들이 Map(Json) 포맷으로 되어 있다. 
     *            하나의 메시지 Json 포맷 형태는 아래와 같다.
     *            // message id: 발신한 메시지의 메시지 아이디
     *            {"mId":"message id 1",                    
     *             // device Ids: 대상 단말별 전달 현황 목록 
     *             "dIds": [                            
     *                       // 퍼블릭 푸시 서버 accept 한 경우
     *                       {
     *                         "dId":"device id 2",
     *                         "ps": 11  
     *                       },
     *                       // 퍼블릭 푸시 Token invalid
     *                       {
     *                         "dId":"device id 2",
     *                         "ps": 13  
     *                       },
     *                       // 퍼블릭 푸시 Token 존재하지 않을 경우
     *                       {
     *                         "dId":"device id 2",
     *                         "ps": 14
     *                       },
     *                       // 퍼블릭 푸시 서버와 통신에러 발생시
     *                       {
     *                         "dId":"device id 2",
     *                         "ps": 19
     *                       },
     *                       // 퍼블릭 푸시 서버 accept, 푸시알림 단말 전달 및 사용자 수신확인 한 경우     
     *                       {
     *                         "dId":"device id 1",       // device id: 대상 단말 아이디 
     *                         "s": 2,                    // state: 메시지 전달 상태가 다음 3가지로 전이된 경우 (2: 사용자 수신확인, 4: 푸시알림 단말에 전달됨, 5: 단말앱 삭제된 상태)
     *                         "nt":111111111,            // notification delivery time: 푸시 알림 전달 시간으로 밀리세컨드 단위( 1970년 1월1일 0시 이후 경과시간) state가 2, 4일 경우 세팅됨
     *                         "dt":222222222             // msg delivery time: 사용자 수신확인 시간으로 밀리세컨드 단위( 1970년 1월1일 0시 이후 경과시간) state가 2일 경우 세팅됨             
     *                         "ps": 11                   // public push state: 퍼블릭 푸시 서버로부터 전달받은 상태  (11: 퍼블릭푸시서버 accept, 12: token not register, 13: invalid token, 19: 퍼블릭푸시서버 통신 에러
     *                       }
     *                     ]
     *            }    
     */
    @Override
    public void onAutoMsgTracked(ArrayList<Map<String, Object>> msgs) {
        
        for(Map<String, Object> msg: msgs) {
            String msgId = (String)msg.get("mId");
            ArrayList<Map<String, Object>> dIds = (ArrayList<Map<String, Object>>)msg.get("dIds");
            
            for(Map<String, Object> device: dIds) {
                String deviceId = (String)device.get("dId");
                
                // check Public State by public push server
                if(device.containsKey("ps")) {
                    int publicPushState = (int)device.get("ps");
                    
                    switch(publicPushState) {
                        case 11: // public push server accept
                            // do your logic
                            break;
                        case 12: // token not register
                            // do your logic
                            break;
                        case 13: // invalid token
                            // do your logic
                            break;
                        case 19: // public push server error
                            // do your logic
                            break;
                        default:                            
                            break;
                    }
                }
                
                // check state by RationalOwl server
                if(device.containsKey("s")) {
                    int state = (int)device.get("s");
                    
                    switch(state) {
                        // user read push message
                        case 2: 
                            // user read time(milliseconds since 1/1/1970)
                            // RationaOwl server deliver message data while app launching by clicking push notification or by clicking app icon
                            long readTime = (long)device.get("dt"); // data delivery time                            
                             
                            if(device.containsKey("nt")) {
                                long notiDeliverTime = (long)device.get("nt");
                            }                            
                            break;
                        // push deliver to the device successfully
                        case 4: 
                            long notiDeliverTime = (long)device.get("nt");
                            break;
                        // user remove app
                        case 5: 
                            // app removed, do your logic here                            
                            break;
                        default:                            
                            break;
                    }
                }
            }            
        }
    }
}
```


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

## 단말앱 동기화 리스너 등록
단말앱 동기화 API 결과를 처리할 리스너를 등록하면 이후 등록해제된 단말앱 아이디 목록을 콜백을 통해 알 수 있다. 

```java
AppServerManager serverMgr = AppServerManager.getInstance();
serverMgr.setDeviceSyncListener(new SimpleDeviceSyncListener());
```


## 유효한 단말앱 아이디 목록 조회 
 시스템푸시를 허용으로 단말앱 자동 동기화 방식을 이용하는 경우 래셔널아울은 삭제된 단말앱까지 동기화하게 된다. 앱서버는 주기적으로 getDeviceIds API를 호출하면 서비스 내 모든 유효한 단말앱 목록을 동기화해야 한다. 한번의 API호출로 최대 20,000개 단말앱 아이디를 조회할 수 있다. 따라서 100만개 단말앱이 있는 서비스의 경우 해당 API를 50회 호출해야 한다.
   
### 유효한 단말앱 아이디 목록 조회 API
 
```java
/**
* 서비스 내 유효한 단말앱 아이디 목록을 요청한다. 
* 결과는 setDeviceSyncListener로 등록한 DeviceSyncListener의 onDeviceIdResult 콜백을 통해  확인한다.
*     
* @param startIndex
*            전체 단말앱 중 fetch 시작 index
* @param fetchSize
*            전체 단말앱 목록 startIndex부터 fetch할 단말앱 아이디 목록 수.
*            최대 20,000
* @return request id
*/
public String getDeviceIds(int startIndex, int fetchSize) {
}
```

### API 사용 예

```java
AppServerManager serverMgr = AppServerManager.getInstance();
serverMgr.getDeviceIds(0, 20000);
```


### 유효한 단말앱 아이디 목록 조회 결과 콜백
onDeviceIdResult 콜백에서 결과를 알 수 있다.

```java
/**
* AppServerManager의 getDeviceIds()API 호출결과 콜백이다.
* 
* @param resultCode
*            Result클래스에 정의된 결과값 상수
* @param resultMsg
*            resultCode의 값에 대한 의미    
* @param totalDeviceSize
*            서비스 내 유효한 단말앱 총 수
* @param startIndex
*            유효한 전체 단말앱 아이디 목록 중 fetch해올 목록의 첫 인덱스
* @param fetchDeviceSize
*            totalDeviceSize 중 startIndex부터 fetchDeviceSize 만큼의 단말앱 아이디를 deviceIds에 fetch 해 온다.
* @param deviceIds
*            fetch 해 온 단말앱 아이디 목록            
*                      
* @param requestId
*            본 콜백 결과를 야기한 getDeviceIds() API 반환값
*/
public void onDeviceIdResult(int resultCode, String resultMsg, int totalDeviceSize, int startIndex, int fetchDeviceSize, ArrayList<String> deviceIds, String requestId) {

}
```

## 단말그룹 내 단말앱 아이디 목록 조회 
 자동 동기화 방식을 이용하는 경우 앱서버는 주기적으로 getDeviceIds API를 호출하여 단말그룹 내 모든 유효한 단말앱 목록을 동기화해야 한다. 한번의 API호출로 최대 20,000개 단말앱 아이디를 조회할 수 있다. 따라서 100만개 단말앱이 있는 단말그룹의 경우 해당 API를 50회 호출해야 한다.
   
### 단말그룹 내 단말앱 아이디 목록 조회 API
 
```java
/**
* 본 API가 호출되면 단말 그룹 내 동기화 작업이 수행되고 단말그룹내 내 유효한 단말앱 아이디 목록이 콜백으로 전달된다. 
* 결과는 setDeviceSyncListener로 등록한 DeviceSyncListener의 onGroupDeviceIdResult 콜백을 통해  확인한다.
* 
* @param deviceGroupId
*            대상 단말그룹 아이디 
* @param startIndex
*            전체 단말앱 중 fetch 시작 index
* @param fetchSize
*            전체 단말앱 목록 startIndex부터 fetch할 단말앱 아이디 목록 수
*            최대 20,000            
* @return request id
*/
public String getSynchronizedGroupDeviceIds(String deviceGroupId, int startIndex, int fetchSize) {
}
```


### 단말그룹 내 단말앱 아이디 목록 조회 결과 콜백
onDeviceIdResult 콜백에서 결과를 알 수 있다.

```java
/**
* AppServerManager의 getDeviceIdsInDeviceGroup()API 호출결과 콜백이다.
* 
* @param resultCode
*            Result클래스에 정의된 결과값 상수
* @param resultMsg
*            resultCode의 값에 대한 의미    
* @param deviceGroupId
*            대상 단말 그룹 아이디       
* @param totalDeviceSize
*            단말그룹 내 유효한 단말앱 총 수
* @param startIndex
*            단말그룹의 전체 단말앱 아이디 목록 중 fetch해올  첫 번째 인덱스
* @param fetchDeviceSize
*            totalDeviceSize 중 startIndex부터 fetchDeviceSize 만큼의 단말앱 아이디를 deviceIds에 fetch 해 온다.
* @param deviceIds
*            fetch 해 온 단말앱 아이디 목록            
*                      
* @param requestId
*            본 콜백 결과를 야기한 getDeviceIdsInDeviceGroup() API 반환값
*/
public void onGroupDeviceIdResult(int resultCode, String resultMsg, String deviceGroupId, int totalDeviceSize, int startIndex, int fetchDeviceSize, ArrayList<String> deviceIds, String requestId) {
}
```





## 단말앱 등록 해제 
 단말앱 수동 동기화 방식에서 삭제 판단일 초과 단말앱에 댛새 앱서버는 unregisterDevices API를 호출하여 래셔널아울 서비스에서도 동기화 하도록 알려야 한다.

  
### 단말앱 등록 해제 API
 

```java
AppServerManager serverMgr = AppServerManager.getInstance();
ArrayList<String> devices = new ArrayList<String>();
devices.add("app id 1");
devices.add("app id 2");
serverMgr.unregisterDevices(devices);
```


### 단말앱 등록 해제 결과 콜백
onDevicesUnregisterResult 콜백에서 결과를 알 수 있다.

```java
class SimpleDeviceSyncListener implements DeviceSyncListener {

    /**
     * AppServerManager의 unregisterDevices()API 호출결과 호출된다.
     * 
     * @param resultCode
     *            Result클래스에 정의된 결과값 상수
     * @param resultMsg
     *            resultCode의 값에 대한 의미
     * @param unregDevices
     *            unregisterDevices() API결과 등록해제된 단말 등록 아이디 목록  resultCode가   Result.RESULT_OK 시 세팅
     * @param requestId
     *            본 콜백 결과를 야기한 unregisterDevices() API 반환값
     */
    public void onDevicesUnregisterResult(int resultCode, String resultMsg, ArrayList<String> unregDevices, String requestId) {

    }
}
```
