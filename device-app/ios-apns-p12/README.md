# 래셔널아울 IOS 설정 가이드

>## Introduction

> P12 인증서를 통해 APNS 푸시알림 설정하는 과정을 설명한다.

아래는 래셔널아울 서비스를 통해 IOS단말앱이 실행되지 않을 때, 단말앱이 P2P 메시지와 다운스트림(푸시) 메시지를 수신하는 과정이다.

![이미지 이름](./img/apns_overview.png)



## 인증서 요청 파일 생성

APNS 서버용 인증서를 생성하기 위해서 먼저 인증서 요청 파일을 생성해야 한다.

1. 맥 우측 상단의 찾기 버튼을 눌러 '키체인 접근'을 검색하여 키체인 접근 앱을 실행시킨다.

![이미지 이름](./img/cert-req1.png)

2. '키체인 접근 > 인증서 지원 > 인증 기관에서 인증서 요청...' 순서로 클릭한다.

![이미지 이름](./img/cert-req2.png)

3. 이메일 주소, 일반 이름을 입력 후 디스크에 저장 옵션 선택 후 계속 버튼 클릭한다.

![이미지 이름](./img/cert-req3.png)

4. 인증서 요청 파일 저장   
'CertificateSigningRequest.certSigningRequest'파일이름으로 저장할 폴더를 지정한다. 인증서 폴더는 개발용 및 상용 서비스용으로 별도 관리 및 저장할 것을 권고한다. 본 문서에서는 개발용 인증서를 기준으로 'dev'폴더에 'CertificateSigningRequest.certSigningRequest'파일이름으로 저장한다.

![이미지 이름](./img/cert-req4.png)


## 단말앱 ID 생성

APNS를 이용하기 위해서는 모든 앱은 고유한App ID를 생성해야 한다. 주의할 점은 App ID 생성과정에서 지정하는 'Bundle ID'를 개발시 XCode에서 지정하는 'Bundle Identifier'와 동일한 값을 지정한다.

1. IOS개발자 페이지(https://developer.apple.com)에 접속하여 우측 상단에Account를 눌러 로그인한다.   
 개발자 페이지에 등록되지 않았다면 먼저 등록해야 한다.

2. 'Certificates, IDs & Profiles' 클릭한다.

![이미지 이름](./img/app-id1.png)

3. 'Identifiers > App IDs'를 클릭하면 기존 생성했던 App ID목록이 나타난다.
4. 우측 상단의 + 를 클릭하여 새 App ID생성한다.

![이미지 이름](./img/app-id2.png)

4. AppID Description에 앱 이름을 입력한다. Ex: RationalOwlSample
5. Explicit App ID에Bundle ID를 입력한다. Ex: com.rationalowl.sample
 - xcode에서 앱 작성시 입력하는  **Bundle Identifier와 동일**한 값

![이미지 이름](./img/app-id3.png)

6. App Services항목에서 Push Notifications항목을 체크한다.
7. 이후 register버튼과 done버튼을 클릭해 App ID생성을 완료한다.

![이미지 이름](./img/app-id4.png)

8. 이후 register버튼과 done버튼을 클릭해 App ID생성을 완료한다.

![이미지 이름](./img/app-id5.png)



## 인증서 생성
앱이 APNS푸시를 이용하기 위해서는 APNS 푸시 발신을 위한 인증서가 필요하다. 
인증서 생성 후 생성한 인증서를 PC에 저장하는 절차는 아래와 같다.

1. App IDs항목에서 방금 생성한 App ID항목을 클릭하면 아래 그림과 같은 화면이 나타난다.

![이미지 이름](./img/cert1.png)

2. Edit버튼을 클릭한다.
3. create certificate...버튼을 클릭한다.      
본 문서에서는 개발용 인증서 생성을 위해 'Development SSL Certificates'란의 'Create Certificate...'버튼을 클릭한다. 실제 상용 서비스용 인증서를 생성하려면 아래 항목인 'Production SSL Certificate'을 선택하면 된다.

![이미지 이름](./img/cert2.png)

4. continue버튼을 클릭한다.

![이미지 이름](./img/cert3.png)

5. Choose File...버튼을 클릭하고 앞서 생성한 인증서 요청 파일인 'CertificateSigningRequest.certSigningRequest'을 선택 후 Continue버튼을 클릭한다.

![이미지 이름](./img/cert4.png)

6. 인증서가 생성되었다. Download버튼을 클릭해서 인증서를 다운 받는다.   

![이미지 이름](./img/cert5.png)

7. 다운받은 인증서를 폴더에 저장한다.
  - 다운받은 인증서를 개발용과 서비스용 별로 구분하여 별도의 폴더에 보관하는 것을 권장
  - 본 가이드에서는 인증서 이름을 'dev_cert.cer'이름으로 저장했다.

![이미지 이름](./img/cert6.png)

## P12 파일 생성
앞서 생성한 인증서(dev_cert.cert)와 개인키를 포함한 p12파일 포맷의 외부 입출력 가능한 파일을 만든다. 


1. 앞서 다운받은 인증서를 더블클릭하면 인증서가 맥OS의 키체인에 포함되고 '키체인 앱'이 실행된다.
2. 키체인 앱에 자신이 방금 생성한 인증서인 Apple Development IOS Push Services인증서의 맨 앞 화살표를 클릭한다.
3. 인증서 화살표가 펼쳐지면 열쇠모양의 개인키가 나타난다. 
4. 인증서와 개인키를 모두 선택 후 마우스 오른쪽을 클릭하면 팝업 메뉴가 나타난다.
5. 팝업 메뉴 중 '2개 항목 내보니기...'를 선택한다.

![이미지 이름](./img/p121.png)


6. 저장하기 화면에서 인증서를 저장했던 폴더에 저장할 p12 파일 이름을 입력한다.
 - 가이드문서에서는 'dev_cert_key'를 입력했다.

![이미지 이름](./img/p122.png)

7. p12 인증서의 암호를 입력한다.

![이미지 이름](./img/p123.png)


8. 키체인 암호를 입력한다.

![이미지 이름](./img/p124.png)

8. 폴더에 'dev_cert_key.p12'이름의 개인키를 포함한 인증서가 생성된 것을 확인할 수 있다.

![이미지 이름](./img/p125.png)



## 래셔널아울 서비스에 P12 파일 업로드하기
앞서 생성한 p12 형식의 인증서(dev_cert_key.p12)를 래셔널아울 관리자 콘솔에 업로드하면 래셔널아울 서비스를 통해 백그라운드의 단말앱에게 APNS를 통한 알림이 가능해진다. 

1. 래셔널아울 관리자 콘솔에 로그인한다.
 - 래셔널아울 서비스 가입시 등록한 이메일과 비밀번호를 입력한다.

![이미지 이름](./img/upload1-1.png)

2. 앞서 생성한 APNS인증서를 이용해서 APNS 푸시를 이용할 모바일 서비스 추가 버튼을 클릭한다.
 - 이미 서비스가 등록했다면 해당 서비스를 클릭해 서비스로 진입한다.
 - 본 가이드 문서에서는 새 모바일 서비스 등록 기준으로 설명한다.

![이미지 이름](./img/upload1-2.png)

3. 모바일 서비스 추가 팝업 화면에 서비스명과 서비스에 대한 설명을 입력 후 추가 버튼을 클릭한다.

![이미지 이름](./img/upload1-3.png)

4. 방금 등록한 서비스 탭이 나타나면 해당 서비스 탭을 클릭한다.

![이미지 이름](./img/upload1-4.png)

5. 새로 등록한 서비스 화면으로 진입 후 단말 현황 메뉴를 클릭한다.

6. 'APNS 등록'버튼을 클릭한다.

![이미지 이름](./img/upload1.png)
7. 인증방식을 'p12 인증서'를 선택한다.
![이미지 이름](./img/upload2.png)

8. APNS 업로드 팝업이 나타나면 '파일선택'버튼을 눌러 앞서 생성한 'dev_cert_key.p12'파일을 선택한다.
 - 인증서를 선택하면 인증서 기본 정보인 파일 이름, 타입, 사이즈 정보가 자동 표시된다.
 - 인증서 타입 필드에 '개발용'을 선택한다.(실제 서비스용은 서비스용 인증서를 만든 후 '상용'을 선택한다.)
 - 인증서 패스워드에는 p12인증서 생성시 입력한 암호를 입력한다.

9. 배포방식을 '개발용' 또는 '배포용' 중 선택한다.
 - 개발단계에서는 '개발용'을 선택한다.
 - 앱스토어에 배포할 경우 '배포용'을 선택한다.

10. Bundle ID를 입력한다.
 - https://developer.apple.com/account/resources/identifiers/list 에서 Bundle ID를 확인 가능하다.
![이미지 이름](./img/upload_bundleId.png)

11. 업로드 버튼을 클릭하여 인증서 상태를 확인한다.
 - 상태가 정상이면 인증서가 정상적으로 업로드되었다.
 - 상태가 비정상으로 나타날 경우 인증서가 잘못 만들어졌다는 의미이며, 본 가이드대로 진행했는지 확인해야 한다.
![이미지 이름](./img/upload3.png)

이로서 래셔널아울 서비스를 통해 IOS 단말앱이 백그라운드일 경우 APNS 알림을 발신 할 준비가 끝났다.

## 프로비저닝 프로파일 생성

앞서 APNS 알림을 발신을 위한 절차가 끝났다. 그러나 IOS 단말앱이 APNS 알림을 수신하기 위한 절차가 남아 있다. 즉, IOS 단말앱에 프로비저닝 프로파일을 생성해야 한다. 앞서 개발용 APNS인증서를 이용하기 위한 개발용 프로비저닝 프로파일 기준으로 설명한다.

1. 웹브라우저로 Apple Developers Center에 로그인 후 'Certificates, Identifiers & Profiles'섹션의 'Provisioning Profiles' 를 클릭한다.
2. 새 프로파일을 만들려면 오른쪽 상단의 + 버튼을 클릭한다.
3. 'Development' 섹션에서 iOS App Development를 선택하고 'Continue'를 클릭한다.

![이미지 이름](./img/profile1.png)

4. 드랍다운 메뉴에서 사용할 APP ID를 선택한다.
 - APNS 개발용 인증서 생성시 입력한 Explicit App ID와 동일한 ID를 선택한다. Ex:com.rationalowl.sample
 - xcode에서 앱 작성시 **Bundle Identifier** 에도 동일한 값을 입력해야 한다.

![이미지 이름](./img/profile3.png)

5. 프로비저닝 프로파일에 포함할 인증서를 선택하고 'Continue'버튼을 누른다.

![이미지 이름](./img/profile4.png)

6. 개발용 프로파일을 적용한 단말앱을 설치할 개발용 단말을 선택한다.

![이미지 이름](./img/profile5.png)

7. 프로비저닝 프로파일 이름을 입력한다.

![이미지 이름](./img/profile6.png)

8. Download를 눌러 프로비저닝 프로파일을 Mac으로 다운로드한다.

![이미지 이름](./img/profile7.png)


9. 다운로드한 프로비저닝 프로파일을 두번 클릭하면 Xcode에 프로비저닝 프로파일이 적용된다.

이제 Xcode로 단말앱 개발시 Bundle ID를 p12 인증서 생성시 입력한 Explicit App ID ( Ex: com.rationalowl.com)와 동일하게 입력하면 해당 단말앱에서 APNS 알림을 수신할 수 있다.
