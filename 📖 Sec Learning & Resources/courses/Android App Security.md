
LINK : [https://app.hextree.io/map/android/android-continent/detail](https://app.hextree.io/map/android/android-continent/detail)

tags:  #android

when you what to learn  android hacking development  you need to learn the basic android developement


Intents and activities are a fundamental concept of Android. We need to understand how they work in order to look for vulnerabilities later. That's why we are going to play around with Intents in our example app.

```java
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://hextree.io/"));
startActivity(browserIntent);
```

The above code example declares our intention (_Intent_) to view (_ACTION_VIEW_) the URL [https://hextree.io/](https://hextree.io/). If we hand over this Intent object to the Android operating system, it will figure out which app can handle this. This means intents can be used to interact with other apps, which makes them one of the most important attack surfaces.

See also `AndroidManifest.xml` of Chrome: [https://chromium.googlesource.com/chromium/src/+/b71e98cdf14f18cb967a73857826f6e8c568cea0/chrome/android/java/AndroidManifest.xml#156](https://chromium.googlesource.com/chromium/src/+/b71e98cdf14f18cb967a73857826f6e8c568cea0/chrome/android/java/AndroidManifest.xml#156)



now our first is to find flag in this  https://github.com/hextreeio/android-challenge1

we have to clone the project  