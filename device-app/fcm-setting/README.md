# 래셔널아울 안드로이드 단말용 FCM 설정 가이드

>## Introduction

> 래셔널아울 서비스를 이용하는 단말앱이 포그라운드일 경우 실시간 데이터 수/발신이 가능하나 안드로이드 단말앱일 경우 백그라운드일 때 FCM(Firebase Cloud Messaging)을 통해 알림 메시지를 수신한다.


아래는 래셔널아울 서비스를 통해 안드로이드 단말앱이 실행되지 않을 때, 단말앱이 P2P 메시지와 다운스트림(푸시) 메시지를 수신하는 과정이다.

![이미지 이름](./img/fcm_overview.png)



## 파이어베이스 콘솔에 프로젝트 생성

[Firebase 콘솔](https://console.firebase.google.com/)에서 안드로이드 단말앱에 푸시알림을 전달하기 위해 프로젝트를 추가해야 한다.

1. '프로젝트 추가'버튼을 클릭한다.

![이미지 이름](./img/add_prj1.png)

2. '프로젝트 추가'창에 프로젝트 이름과 국가/지역을 입력한다.

![이미지 이름](./img/add_prj2.png)

3. 'Android 앱에 Firebase 추가' 클릭

![이미지 이름](./img/add_prj3.png)

4. 안드로이드 패키지명과 앱닉네임을 입력하고 '앱등록' 버튼을 클릭한다.

![이미지 이름](./img/add_prj4.png)

5. 'google-services.json'파일을 다운로드 후 안드로이드 스튜디오 앱루트 디렉토리에 카피한다. 

![이미지 이름](./img/add_prj5.png)


6. 프로젝트 레벨 'build.gradle' depencies에 google-services를 추가한다.

최신 안드로이드 단말앱 라이브러리는 <a href="https://guide.rationalowl.com/library" target="_blank">Android 샘플앱</a>
의 프로젝트레벨의 build.gradle을 참조한다.

아래는 2019년 12월 기준 샘플앱의 프로젝트레벨 build.gradle의 dependencies 부분이다.

```java
dependencies {
    ...

    classpath 'com.android.tools.build:gradle:3.3.0'
    classpath 'com.google.gms:google-services:4.1.0'

    ...
}
```

7. 앱 레벨 'build.gradle' depencies에 FCM 라이브러리를 추가한다.

최신 안드로이드 단말앱 라이브러리는 <a href="https://guide.rationalowl.com/library" target="_blank">Android 샘플앱</a>
의 앱레벨의 build.gradle을 참조한다.

// FCM library 환경을 그대로 카피하여 적용하면 된다.

아래는 2019년 12월 기준 샘플앱의 앱레벨 build.gradle의 dependencies 부분이다.
최근 안드로이드 샘플앱의 FCM library뿐 아니라 RationalOwl library 및 RationalOwl using library를 그대로
카피하면 된다.

```java
dependencies {
    ...

    // RationalOwl library
    implementation 'com.rationalowl.minerva.client.android:rationalowl-android:1.1.7'
    // RationalOwl using library
    implementation 'android.arch.lifecycle:extensions:1.1.0'

    // FCM library
    implementation 'com.google.firebase:firebase-core:16.0.6'
    implementation 'com.google.firebase:firebase-messaging:17.3.4'

    ...
}
```



## FCM 안드로이드 단말앱 적용

FCM 푸시 알림을 안드로이드 단말앱에 적용하기 위해서 를 처리하기 위해서는 FirebaseMessagingService 구현해야 한다.

1. FCM 단말 토큰 생성시 호출되는 서비스인 onNewToken(String token) 메소드를 구현한다.
    - 해당 함수내에서 setDeviceToken() API를 호출해야 한다.

2. 커스텀(Customizable Push)를 처리하는 onMessageReceived(RemoteMessage remoteMessage)를 구현한다.
    - 해당 함수내에서 enableNotificationTracking() API를 호출해야 한다.

아래는 2019년 12월 기준 샘플앱이다.


```java
public class MyFirebaseMessagingService extends FirebaseMessagingService {

    private static final String TAG = "MyFirebaseMsgService";


    /**
     * Called if InstanceID token is updated. This may occur if the security of
     * the previous token had been compromised. Note that this is called when the InstanceID token
     * is initially generated so this is where you would retrieve the token.
     */
    @Override
    public void onNewToken(String token) {
        Logger.debug(TAG, "onNewToken token: " + token);
        // just call setDeviceToken() API
        MinervaManager minMgr = MinervaManager.getInstance();
        minMgr.setDeviceToken(token);

        // there's no need to do anything else
    }


    /**
     * Called when message is received.
     *
     * @param remoteMessage Object representing the message received from Firebase Cloud Messaging.
     */
    // [START receive_message]
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        Logger.debug(TAG, "onMessageReceived enter");
        Map<String, String> data = remoteMessage.getData();

        // set notification  delivery tracking
        MinervaManager minMgr = MinervaManager.getInstance();
        minMgr.enableNotificationTracking(data);


        // TODO(developer): Handle FCM messages here.
        // Not getting messages here? See why this may be: https://goo.gl/39bRNJ
        Log.d(TAG, "From: " + remoteMessage.getFrom());

        showCustomNotification(data);
    }


    /**
     * Create and show user defined custom notification such as image/rich notification.
     *
     * @param data FCM message body.
     */
    private void showCustomNotification(Map<String, String> data) {
        Intent intent = new Intent(this, MainActivity.class);
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0 /* Request code */, intent,
                PendingIntent.FLAG_ONE_SHOT);

        Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
        NotificationCompat.Builder notificationBuilder =
                new NotificationCompat.Builder(this, "defaultChannelId")
                        .setSmallIcon(R.drawable.icon)
                        .setContentTitle("content title")
                        .setContentText("content text")
                        .setAutoCancel(true)
                        .setSound(defaultSoundUri)
                        .setContentIntent(pendingIntent);

        NotificationManager notificationManager =
                (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);

        // Since android Oreo notification channel is needed.
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel("defaultChannelId",
                    "FCM Channel",
                    NotificationManager.IMPORTANCE_DEFAULT);
            notificationManager.createNotificationChannel(channel);
        }

        notificationManager.notify(0 /* ID of notification */, notificationBuilder.build());
    }
}
```

이로써 안드로이드 단말앱 설정이 끝났다.

## FCM 서버키 등록

1. [Firebase 콘솔](https://console.firebase.google.com/)에서 이전단계에서 생성한 프로젝트를 클릭한다.

![이미지 이름](./img/add_prj1.png)


2. '프로젝트 설정' 버튼을 클릭한다.

![이미지 이름](./img/server_key_1.png)

3. '클라우드 메시징' 탭을 누르면 서버키 값이 나타나는데 해당 값을 카피한다.

![이미지 이름](./img/server_key_2.png)

4. 래셔널아울 콘솔에 로그인한다.

![이미지 이름](./img/server_key_3.png)

5. '+서비스'버튼을 클릭하여 서비스 이름과 서비스 설명을 입력후 추가 버튼을 클릭한다.

![이미지 이름](./img/server_key_4.png)

6. 서비스 목록에 추가된 서비스의 '바로가기' 버튼을 클릭한다.

![이미지 이름](./img/server_key_5.png)

7. 'FCM 서버키 등록' 버튼을 클릭한다.

![이미지 이름](./img/server_key_6.png)


8. 앞서 Firebase 콘솔에서 카피한 서버키를 입력 후 '업로드' 버튼을 클릭한다.

![이미지 이름](./img/server_key_7.png)

9. FCM 서버키가 정상 등록된 것을 확인한다.

![이미지 이름](./img/server_key_8.png)

이로서 안드로이드 단말앱이 백그라운드일 경우 앱서버로부터의 다운스르트림 메시지나 다른 단말앱으로부터의 P2P 메시지를 수신시 FCM 을 통한 알림 메시지를 수신할 수 있다.