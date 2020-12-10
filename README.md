## Location and Context APIs Tutorial

Sample link: [editor on GitHub](https://github.com/aganovia/ParkingAppTutorial/edit/main/README.md)'
# Header 1 ## Header 2 ### Header 3
``` highlighted code block```
[Link](url) and ![Image](src)
- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

# Overview

Our group researched [Location and Context APIs](https://developers.google.com/location-context/) on Android. In order to develop proficiency in this area, we focused specifically on the location aspect of it, since it offers the most functionality and is the most commonly implemented in applications.

We created a simple Parking app that is intended to show users their current location, allow them to save the location, and send the location to Google Maps, or copy it to their clipboard. Having your parking spot saved in your phone can make life a lot easier!

The final application will look like this:

![Image](https://i.imgur.com/zYNCY2d.png)  ![Image](https://i.imgur.com/0KSkPlx.png)

# Getting Started

You will need Android Studio for this demonstration. The app will be written in Kotlin, and will support API version 19 (KitKat) and later. (include screenshot)

You will also need to add the following dependencies to your project's build.gradle:

```
implementation 'com.google.android.gms:play-services-location:17.0.0'
implementation 'com.google.android.libraries.places:places:2.4.0'
```

The first dependency is Google Play Location Services. This is what allows us to create a Fused Location Provider Client, which is how we retrieve the device's last known location and latitude/longitude coordinates.

The second dependency comes from the Google Places API. This provides our app with backend geographic data that makes it possible to convert our coordinates into an address. This will be done through Android's Geocoder class. Without this dependency, the Geocoder would not function.

Also make sure that Google is included in your project's repositories (also in the build.gradle) if it isn't already:

```
ext.kotlin_version = "1.3.72"
    repositories {
        google()
        ...
    }
```

Don't forget to sync your gradle files in your project before testing your app.

Finally, in your project's manifest, make sure to include these permissions:

```
<uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```

These are what allow our app to access location data. It will need an internet connection and access to the device's location.

# Instructions

# Conclusions

