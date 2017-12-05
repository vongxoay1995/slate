BCT Android SDK
==========================

BCT Android SDK is a library to support build call apps with Twilio.


Setup Project
=====

*For a working implementation of this project see the `sample/` folder.*

   1. Coppy folder in path `AndroidSDK\callsdk\build\repo`. Paste into folder libs in my project by following path: `[Project_name]\app\build\libs`.
  
   2. At your project, which you you're development,add rules to your root-level build.gradle file,
    to include the Maven repository local contains the BCT Android SDK :
	
	```
    allprojects {
	  repositories {
		jcenter()
	    maven{
		  url "$buildDir/libs"
	    }
	  }
    }
 or...
allprojects {
	  repositories {
		jcenter()
		maven{
		  url "../app/build/libs"
		}
	  }
    }
```
   3. Then, in your module Gradle file (usually the app/build.gradle), add the dependency below:
  
	```
    dependencies {
          compile ('com.bct:android-sdk:0.5.0'){
             transitive=true
         }
    }
	```
	
Setup Firebase
=====
 1. Firebase 
   The BCT Android SDK uses Firebase Cloud Messaging push notifications to let your application know when it is receiving an incoming call. If you want your users to receive incoming calls,
   you’ll need to enable FCM in your application.
   Follow the steps under Use the Firebase Assistant in the Firebase Developers Guide. Once you connect and sync to Firebase successfully, you will be able to download the google-services.json for your application.
   Login to Firebase console and make a note of generated Server API Key in your notepad. You will need them in the next step.
   
   
   Make sure the generated google-services.json is downloaded to the app directory of the your project to replace the existing app/google-services.json stub json file. If you are using the Firebase plugin make sure to remove the stub google-services.json file first.
   As a final step re-run the application from Android Studio to ensure the APK now has the latest google-services.json file.
   You can refer to the following link : https://firebase.google.com/docs/android/setup 
   
   
2. Add a Push Credential using your FCM Server API Key

   You will need to store the FCM Server API Key with BCT Server so that we can send push notifications to your app on your behalf. Once you store the API Key with BCT server, it will get assigned a Push Credential SID so that you can later specify which key we should use to send push notifications.
   You must send my FCM Server API Key to BCT Server.   
3. Create class SdkService 
   code  below:
   ```public class SDKService extends IntentService {
    ...
    public String fcmToken;
    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        fcmToken= FirebaseInstanceId.getInstance().getToken();
        BCTHelper.getInstance().retrieveCapabilityToken(fcmToken);
    }
    ...
   }```
 3. Then, in your module Gradle file (usually the app/build.gradle), add the dependency below:

	```
    dependencies {
          compile 'com.google.firebase:firebase-messaging:11.6.0'
    }
	```
	
Usage
===============
 1. In your class MyService  extends IntentService. In method onHandleIntent(), You write the following commands:
        fcmToken= FirebaseInstanceId.getInstance().getToken();
        BCTHelper.getInstance().retrieveCapabilityToken(fcmToken);
  
 2. In your class Service to receive notifications from firebase. You need to implement by VoiceHandleMessage.Listener and write the following commands:
 
	```
    public void onCreate() {
	     ...
        voiceHandleMessage = new VoiceHandleMessage();
        voiceHandleMessage.setListener(this);
        super.onCreate();
    }
    public void onMessageReceived(RemoteMessage remoteMessage) {
        ...
        if (remoteMessage.getData().size() > 0){
            Map<String, String> data = remoteMessage.getData();
            voiceHandleMessage.handleMessage(getApplicationContext(), data);
        }
    }
	```
	
	BCT gives you two methods to process: 
	```void onCallInvite(BCTCallInvite bctCallInvite);```
	```void onError(BCTMessageException bctMessageException);```
 3. You can regiter identity by following api: 
    ``` BCTHelper.getInstance().postIdentity(appKey, appSecret, clientId);```
 4. You need to implement BCTCallListener.Listener to listen to events:
     . when client connect
	``` void onConnected(BCTCall bctCall); ```
	 . when client connected failed
	``` void onConnectFailure(BCTCall bctCall, BCTCallException exception); ```
	 . when client disconnect
      ```void onDisConnected(BCTCall bctCall, BCTCallException exception);  ```
 5. When incomming call, you will use the following method to connect:
     ``` void phoneAccept(CallInvite callInvite, Listener listener); ```
 6. When reject call, you will use the following method to reject call:
     ``` void phoneReject(CallInvite callInvite); ```
 7. When end call, you will use the following method to disconnect call:
    ```  void phoneDisconect(Call activeCall) ; ```
 8. When unregister BCT Android SDK, you need called to method:
     ``` void unregisterBCTCall(); ```
