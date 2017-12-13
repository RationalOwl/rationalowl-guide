# Rationalowl Minerva Node Server SDK

> Rationalowl Minerva Node Server SDK는 Node.js로 구성된 서버에서 Minerva 실시간 메시지 전송을 이용할 수 있는 라이브러리입니다. 

## 목차

- [Introduction](#Introduction)
- [설치](#설치)
  - [NPM](#NPM)
  - [YARN](#YARN)
- [앱 서버 등록/해제](#앱 서버 등록/해제)
  - [registerAppServer](#registerAppServer)
  - [unregisterAppServer](#unregisterAppServer)
- [디바이스 그룹](#디바이스 그룹)
  - [createDeviceGroup](#createDeviceGroup)
  - [addDeviceGroup](#addDeviceGroup)
  - [subtractDeviceGroup](#subtractDeviceGroup)
  - [deleteDeviceGroup](#deleteDeviceGroup)
- [메시지 발신](#메시지 발신)
  - [sendUnicastMsg](#sendUnicastMsg)
  - [sendMulticastMsg](#sendMulticastMsg)
  - [sendBroadcastMsg](#sendBroadcastMsg)
  - [sendGroupMsg](#sendGroupMsg)
- [메시지 수신](#메시지 수신)
  - [addReceivedUpstreamMsgListener](#addReceivedUpstreamMsgListener)
  - [removeReceivedUpstreamMsgListener](#removeReceivedUpstreamMsgListener)

## Introduction

- 메시지 서버에 수신/발신 API 제공
- Unicast, Multicast, Broadcast API 제공
- Upstream 이벤트 제어 가능

## 설치

### NPM

```sh
npm install --save minerva-node-sdk
```

### YARN

```sh
yarn add minerva-node-sdk
```

## 앱 서버 등록/해제

앱 서버를 등록하여 메시지 수신/발신에 따른 추가적인 서비스를 구현 할 수 있습니다.
서버 등록에 성공하면 서버 등록 아이디가 발급되고 이후 재 등록할 필요가 없습니다.

### registerAppServer

```js
import { AppServerManager } from "minerva-node-sdk";
const server = await AppServerManager.getInstance()
    .registerAppServer(serviceId, regName, gateHost, gatePort);
/* server
{ resultCode: 1,
  resultMsg: '작업이  성공 했습니다.',
  appServerRegId: 'def829b853d046779e2227bdd091653c:null' }
 */
```

앱 서버를 등록합니다. 반드시 처음에 불러야 하는 api 입니다.

- Paremeter
  - `serviceId` - (string) 웹 콘솔에서 등록된 서비스의 아이디
  - `regName` - (string) 등록할 앱 서버의 등록 이름
  - `gateHost` - (string) *Optional* 게이트웨이 서버의 호스트 주소 (Default: 'gate.rationalowl.com')
  - `gatePort` - (number) *Optional* 게이트웨이 서버의 포트 (Default: 9081)
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1, 이미 등록된 경우 = -102)
    - `resultMsg` - (string) 결과 메시지
    - `appServerRegId` - (string) 등록된 앱 서버의 등록 아이디

### unregisterAppServer

```js
const unRegServer = await AppServerManager.getInstance()
    .unregisterAppServer(serviceId, serverRegId);
/* unRegServer
{ resultCode: 1, resultMsg: '작업이  성공 했습니다.' }
 */
```

등록된 앱 서버를 제거합니다.

- Paremeter
  - `serviceId` - (string) 웹 콘솔에서 등록된 서비스의 아이디
  - `serverRegId` - (string) 앱 서버 등록 아이디
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지

## 디바이스 그룹

디바이스들을 하나의 그룹으로 설정하여 그룹에 대해 메시지를 보낼 수 있습니다.

### createDeviceGroup

```js
const newGroup = await AppServerManager.getInstance()
    .createDeviceGroup(groupName, groupDesc, deviceList);
/* newGroup
{ resultCode: 1,
  resultMsg: '작업이  성공 했습니다.',
  deviceGrpId: 'dfa29b3b6bbc41d391666a24a8744c06',
  deviceGrpName: 'groupName',
  deviceSize: 5,
  desc: 'groupDesc',
  failedDevcies: [] }
 */
```

디바이스 그룹을 등록합니다.

- Paremeter
  - `groupName` - (string) 등록할 그룹의 이름
  - `groupDesc` - (string) 등록할 그룹의 설명
  - `deviceList` - (string[]) 등록할 그룹에 속할 디바이스의 아이디 리스트
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지
    - `deviceGrpId` - (string) 등록된 그룹의 아이디
    - `deviceGrpName` - (string) 등록된 그룹의 이름
    - `deviceSize` - (string) 등록된 그룹에 속한 디바이스 개수
    - `desc` - (string) 등록된 그룹의 설명
    - `failedDevcies` - (string[]) 그룹에 디바이스 등록중 실패한 디바이스 아이디 리스트

### addDeviceGroup

```js
const addGrpMsg = await AppServerManager.getInstance()
    .addDeviceGroup(groupId, deviceList);
/* addGrpMsg
{ resultCode: 1,
  resultMsg: '작업이  성공 했습니다.',
  deviceGrpId: '43218a6cc0944f23b702620b4becdd17',
  totalDeviceSize: 10,
  addedDeviceSize: 5,
  failedDevcies: [] }
 */
```

디바이스 그룹에 디바이스를 등록합니다.

- Paremeter
  - `groupId` - (string) 디바이스 그룹 아이디
  - `deviceList` - (string[]) 추가할 디바이스의 아이디 리스트
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지
    - `deviceGrpId` - (string) 디바이스 그룹 아이디
    - `totalDeviceSize` - (string) 디바이스 그룹에 속한 디바이스 개수
    - `addedDeviceSize` - (string) 추가된 디바이스의 개수
    - `failedDevcies` - (string[]) 그룹에 디바이스 등록중 실패한 디바이스 아이디 리스트

### subtractDeviceGroup

```js
const subGrpMsg = await AppServerManager.getInstance()
    .subtractDeviceGroup(groupId, deviceList);
/* subGrpMsg
{ resultCode: 1,
  resultMsg: '작업이  성공 했습니다.',
  deviceGrpId: '68379aa276a84935b9325319f2c81e23',
  totalDeviceSize: 5,
  subtractDeviceSize: 5,
  failedDevcies: [] }
 */
```

디바이스 그룹에 디바이스를 등록합니다.

- Paremeter
  - `groupId` - (string) 디바이스 그룹 아이디
  - `deviceList` - (string[]) 삭제할 디바이스의 아이디 리스트
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지
    - `deviceGrpId` - (string) 디바이스 그룹 아이디
    - `totalDeviceSize` - (string) 디바이스 그룹에 속한 디바이스 개수
    - `subtractDeviceSize` - (string) 삭제한 디바이스의 개수
    - `failedDevcies` - (string[]) 그룹에 디바이스 등록중 실패한 디바이스 아이디 리스트

### deleteDeviceGroup

```js
const deleteGrpMsg = await AppServerManager.getInstance()
    .deleteDeviceGroup(groupId);
/* deleteGrpMsg
{ resultCode: 1,
  resultMsg: '작업이  성공 했습니다.',
  deviceGrpId: '67c5a0e242804eb18fe3bf82799d164c' }
 */
```

디바이스 그룹을 제거합니다.

- Paremeter
  - `groupId` - (string) 제거할 디바이스 그룹 아이디
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지
    - `deviceGrpId` - (string) 제거된 디바이스 그룹 아이디

## 메시지 발신

메시지를 발신 할 수 있습니다.

Unicast : 한대의 디바이스에 메시지를 보냅니다.

Multicast : 여러대의 디바이스에 메시지를 보냅니다. 요금제에 따라 한번에 보낼 수 있는 디바이스 대수의 한계가 있습니다.

Broadcast : 서비스에 등록된 모든 디바이스에 메시지를 보냅니다.

GroupMessage : 디바이스 그룹에 속한 모든 디바이스에 메시지를 보냅니다.

### sendUnicastMsg

```js
const unicastMsg = await AppServerManager.getInstance()
    .sendUnicastMsg(data, notiMsg, deviceRegId, isSupportMsgQ);
/* unicastMsg
{ resultCode: 1, resultMsg: '작업이  성공 했습니다.' }
 */
```

Unicast 메시지를 보냅니다.

- Paremeter
  - `data` - (string) 전달할 메시지 데이터
  - `notiMsg` - (string) 단말앱이 구동중이 아닐때 표시할 알림 문자 (null이면 data를 알림 창에 표시)
  - `deviceRegId` - (string) 메시지를 전달 할 대상 단말 앱의 단말 등록 아이디
  - `isSupportMsgQ` - (boolean) 단말앱이 비활성일 때 미 전달 메시지를 큐잉할지 여부
    - `false` 단말이 비활성으로 메시지 수신하지 않더라도 재발송하지 않는다.
    - `true` 미전달 메시지를 메시징 서버에서 메시지 큐잉기간(디폴트 7일)동안 큐잉하고 있다가 단말상태가 활성상태가 되면 미전달 메시지를 단말 앱에 전달한다.
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지

### sendMulticastMsg

```js
const multicastMsg = await AppServerManager.getInstance()
    .sendMulticastMsg(data, notiMsg, deviceRegIds, isSupportMsgQ);
/* multicastMsg
{ resultCode: 1, resultMsg: '작업이  성공 했습니다.' }
 */
```

Multicast 메시지를 보냅니다.

- Paremeter
  - `data` - (string) 전달할 메시지 데이터
  - `notiMsg` - (string) 단말앱이 구동중이 아닐때 표시할 알림 문자 (null이면 data를 알림 창에 표시)
  - `deviceRegIds` - (string[]) 메시지를 전달 할 대상 단말 앱의 단말 등록 아이디 리스트
  - `isSupportMsgQ` - (boolean) 단말앱이 비활성일 때 미 전달 메시지를 큐잉할지 여부
    - `false` 단말이 비활성으로 메시지 수신하지 않더라도 재발송하지 않는다.
    - `true` 미전달 메시지를 메시징 서버에서 메시지 큐잉기간(디폴트 7일)동안 큐잉하고 있다가 단말상태가 활성상태가 되면 미전달 메시지를 단말 앱에 전달한다.
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지

### sendBroadcastMsg

```js
const broadcastMsg = await AppServerManager.getInstance()
    .sendBroadcastMsg(data, notiMsg, isSupportMsgQ);
/* broadcastMsg
{ resultCode: 1, resultMsg: '작업이  성공 했습니다.' }
 */
```

Broadcast 메시지를 보냅니다.

- Paremeter
  - `data` - (string) 전달할 메시지 데이터
  - `notiMsg` - (string) 단말앱이 구동중이 아닐때 표시할 알림 문자 (null이면 data를 알림 창에 표시)
  - `isSupportMsgQ` - (boolean) 단말앱이 비활성일 때 미 전달 메시지를 큐잉할지 여부
    - `false` 단말이 비활성으로 메시지 수신하지 않더라도 재발송하지 않는다.
    - `true` 미전달 메시지를 메시징 서버에서 메시지 큐잉기간(디폴트 7일)동안 큐잉하고 있다가 단말상태가 활성상태가 되면 미전달 메시지를 단말 앱에 전달한다.
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지

### sendGroupMsg

```js
const groupMsg = await AppServerManager.getInstance()
    .sendGroupMsg(data, notiMsg, deviceGroupId, isSupportMsgQ);
/* groupMsg
{ resultCode: 1, resultMsg: '작업이  성공 했습니다.' }
 */
```

Broadcast 메시지를 보냅니다.

- Paremeter
  - `data` - (string) 전달할 메시지 데이터
  - `notiMsg` - (string) 단말앱이 구동중이 아닐때 표시할 알림 문자 (null이면 data를 알림 창에 표시)
  - `deviceGroupId` - (string) 메시지를 전달할 디바이스 그룹 아이디
  - `isSupportMsgQ` - (boolean) 단말앱이 비활성일 때 미 전달 메시지를 큐잉할지 여부
    - `false` 단말이 비활성으로 메시지 수신하지 않더라도 재발송하지 않는다.
    - `true` 미전달 메시지를 메시징 서버에서 메시지 큐잉기간(디폴트 7일)동안 큐잉하고 있다가 단말상태가 활성상태가 되면 미전달 메시지를 단말 앱에 전달한다.
- Return
  - `Promise`
    - `resultCode` - (number) 결과 코드 (성공 = 1)
    - `resultMsg` - (string) 결과 메시지


## 메시지 수신

단말로 부터 전달된 메시지를 수신할 수 있습니다.

### addReceivedUpstreamMsgListener

```js
function listener(res){
    console.log(res);
}
AppServerManager.getInstance().addReceivedUpstreamMsgListener(listener);
```

단말에서 앱 서버로 전송되는 Upstream 메시지에 대한 리스너를 추가합니다.

- Paremeter
  - `listener` - (function) Upstream 메시지에 대한 리스너
- Return
  - `sender` - (string) 메시지를 보낸 단말 아이디
  - `serverTime` - (number) 메시지 보낸 시간
  - `data` - (string) 메시지

### removeReceivedUpstreamMsgListener

```js
function listener(res){
    console.log(res);
}
AppServerManager.getInstance().addReceivedUpstreamMsgListener(listener);
AppServerManager.getInstance().removeReceivedUpstreamMsgListener(listener);
```

등록된 Upstream 메시지에 대한 리스너를 제거합니다.

- Paremeter
  - `listener` - (function) 제거할 리스너
