---
title: "PiFire on Android"
permalink: /android
sort: 6
---
## Adding the Web App to your Homescreen using Chrome on your Android Phone

If you are an Android person, you are likely to be using Chrome on your phone and can not only setup a link to the web-app on your homescreen, but it also makes the interface look like a native application.  Pretty cool right?  Well here's how you set this up, it's very simple.  

First, navigate to the instance of the application in your Chrome browser on your phone.  Remember, it's as easy as going to the IP address that was assigned to you device.  Then, from the Chrome drop-down menu in the upper right of the screen, select "Add to Home screen".  

![Add to Home screen in Chrome](/img/webui/PiFire-Chrome-00.jpg)

Then, when the new dialog box pops up, you will have the opportunity to rename the application, or keep the default.

![Rename the application](/img/webui/PiFire-Chrome-01.jpg)

And there you have it, you've not only created a quick link to your web-app, but you've also created a pseudo application at the same time.

![App on Homescreen](/img/webui/PiFire-Chrome-02.jpg)

## Android Application in Development

Github user [@weberbox](https://github.com/weberbox) has developed an Android application for PiFire.  It's actively under development.  [(https://github.com/weberbox/PiFire-Android)](https://github.com/weberbox/PiFire-Android)

![Android Dash](/img/android/PiFire-Android-00.png)

From Weberbox (modified slightly for brevity):

_For mobile development PiFire Android v1.3.1 is designed for the current state of the PiFire development branch._

_There still may be some bugs here and there as I cannot test every device but I think it is stable enough. There is an error reporting feature built in if the app crashes it is up to the user if they want to share a crash report or not. It can also be disabled in app settings if desired._

_Releases can be found here: 
https://github.com/weberbox/PiFire-Android/releases_

_There are currently two versions that can be downloaded:_

* _Github Firebase – This supports push notifications directly to the application from PiFire. It was redesigned so it no longer requires custom compiling the Android source and signing up for Firebase etc. This uses a hosted aws server to facilitate communication from a PiFire Server(grill) to Google Firebase then to the Android Client. You only need to enable it from the app once you have connected the app to a PiFire server. The middle service is currently setup using a free tier of hosting and with the current user base of PiFire it should remain in the free tier. If PiFire ever grows large this may need to be reworked to avoid a monthly cost on my part. This version requires the device to have Google play services installed._

* _Github NonFirebase – This version does not support mobile push notifications in the Android app. Pushbullet, Pushover, IFTTT, etc are still supported by the server but PiFire Android will not receive push notifications directly. This version does not require Google play services installed._

_There is also one more version that can be self compiled in case anyone wants to do that. It should be straight forward. Just download the source and import the project to Android Studio. There usually will be a few prompts to download an NDK and/or SDK. This option would require some additional work for push notifications as it does not include all the needed Google play files or server links._
