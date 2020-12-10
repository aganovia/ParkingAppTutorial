# Parking App Tutorial
## Overview

Our group researched [Location and Context APIs](https://developers.google.com/location-context/) on Android. In order to develop proficiency in this area, we focused specifically on the location aspect of it, since it offers the most functionality and is the most commonly implemented in applications.

We created a simple Parking app that is intended to show users their current location, allow them to save the location, and send the location to Google Maps, or copy it to their clipboard. Having your parking spot saved in your phone can make life a lot easier!

The final application will look like this:

![Image](https://i.imgur.com/zYNCY2d.png)  ![Image](https://i.imgur.com/Ion8Wya.png)

The app makes use of Android's Navigation Component and Recycler View.

## Getting Started

You will need Android Studio for this demonstration. The app will be written in Kotlin, and the minimum SDK will be set to API 19: Android 4.4 (KitKat).

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

## Instructions

1) Because our app uses Android's Navigation component, we will be writing our location code in a fragment called MainFragment. The same code will work in an Activity with minor changes (for instance, areas that say "this.context" or "this.context as Activity" should be converted to "this").

The onCreateView function will look like so:

```
override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        setHasOptionsMenu(true)
        return inflater.inflate(R.layout.fragment_main, container, false);
    }
```

2) Next, create the following global variables at the top of your MainFragment:

```
val RequestPermissionCode = 1
var mLocation: Location? = null
private lateinit var fusedLocationProviderClient: FusedLocationProviderClient
private lateinit var mLocationRequest: LocationRequest
var latitude = 0.0
var longitude = 0.0
var time = ""
var date = ""
var featureName = ""
var state = ""
var addressLine = ""
```

Most of these are self explanatory as to the values they will hold. RequestPermissionCode, mLocation, fusedLocationProviderClient, and mLocationRequest will all be involved in the process of retrieving the device's last known location, and asking for location updates from the device. We will see how they are implemented in future steps.

3) The onViewCreated function will be where we do the majority of the work. In this function, start by instantiating the fusedLocationProviderClient, and import as necessary: 

```
fusedLocationProviderClient = LocationServices.getFusedLocationProviderClient(this.context as Activity)
```

4) Directly underneath that, set up location requests with the following code:

```
// location requests
mLocationRequest = LocationRequest.create()
mLocationRequest.interval = 1000
mLocationRequest.fastestInterval = 1000
mLocationRequest.priority = LocationRequest.PRIORITY_HIGH_ACCURACY
var mLocationCallback = LocationCallback()
```

The reason we implement location requests is so that our app is aware when the device's location changes. Without this, we would be able to retrieve the device's last known location, but once it changes, it would not have the most accurate updated location available to it. You can read more about Android location requests [here] (https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest).

In our example, we set the interval at which location requests are made to 1 second. This is for the purposes of demonstrating our app in real time; realistically, an application may set it to 5+ seconds depending on how often they believe the user's location will change. 

The fastest interval line isn't required for our purposes, but in a real app, this would set the interval for updating location based on other apps. For instance, if the device's location can be retrieved more quickly from a different application, it will do so with this interval.

We set the priority of our requests to high accuracy because we want the most accurate location data available, and for our app to work in real-time. Other options include PRIORITY_NO_POWER (best accuracy possible with no additional power consumption), PRIORITY_LOW_POWER (to request "city" level accuracy), and PRIORITY_BALANCED_POWER_ACCURACY (to request "block" level accuracy).

A location callback is created to be used in conjunction with our request for location updates, which will be shown in following steps.

5) We would like our app to request location permissions once it starts, if those permissions aren't already granted.  This code will be fairly straight-forward. Directly underneath the previous code you added, add this block:

```
// permissions check required before location updates can be requested
        if (ActivityCompat.checkSelfPermission(
                this.context as Activity,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            requestPermission()
        }
```

Then create the requestPermission() function outside of the onViewCreated():

```
private fun requestPermission() {
        // request location permissions
        ActivityCompat.requestPermissions(
            this.context as Activity,
            arrayOf(Manifest.permission.ACCESS_FINE_LOCATION),
            RequestPermissionCode
        )
    }
```

Permissions must be requested in order for the app to function. It is also reassuring to users to explicitly asked for location permissions before retrieving their data.

6) Next, we will implement a function that retrieves the device's last known location:

```
fun getLastLocation(latvalue: TextView, longvalue: TextView, addvalue: TextView, addextravalue: TextView, mLocationCallback: LocationCallback) {
        // get the device's last location
        if (ActivityCompat.checkSelfPermission(
                this.context as Activity,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            requestPermission()
        } else {
            fusedLocationProviderClient.lastLocation
                .addOnSuccessListener { location: Location? ->
                    mLocation = location
                    // request location update to get most current location
                    LocationServices.getFusedLocationProviderClient(this.context as Activity).requestLocationUpdates(mLocationRequest, mLocationCallback, null)
                    if (location != null) {
                        // retrieve values
                        latitude = mLocation?.latitude!!
                        longitude = mLocation?.longitude!!
                        time = android.text.format.DateFormat.getTimeFormat(this.context).format(location.time)
                        date = android.text.format.DateFormat.getDateFormat(this.context).format(location.time)

                        // Update text on-screen to reflect location info
                        latvalue.text = latitude.toString()
                        longvalue.text = longitude.toString()
                        getAddress(latitude, longitude)
                        addvalue.text = "$addressLine, $state"
                        addextravalue.text = "$featureName"
                    }
                    else {
                        Log.d("CIS357Project", "No location found")
                    }
                }
        }
    }
```

The first part of the function checks again to make sure that permissions are granted. Although we already added one permissions check, you will notice that if you get rid of that "if" check, Android Studio will notify you that what you're trying to do with the fusedLocationProviderClient requires location permissions from the user, and will strongly suggest that you add it. Although it may not be entirely necessary, it's a good safety precaution to have in place with most location-related operations.

This is where our fusedLocationProviderClient comes in. First, we request location updates using our location request and callback to ensure that we have access to the most accurate location information available to us. Next, we check to make sure that the location we're retrieving is not null. If not, we retrieve its latitude, longitude, the time at which it was retrieved, and the date on which it was retrieved and store this in our global variables. Next, we simply set the text of our TextViews that we passed in so this location data is communicated to the user's screen.

We will use this getLastLocation() function each time our "Update Location" button is pressed. Add this click listener to your onViewCreated() code, and pass the text views that display latitude, longitude, the first address line, and the second address line, as well as our location callback:

```
updateButton.setOnClickListener { v ->
            getLastLocation(latvalue, longvalue, addvalue, addextravalue, mLocationCallback)
        }
```

You will notice that we don't have a getAddress function yet. We will add that in the next step.

7) Add the code for the getAddress function:

```
private fun getAddress(lat: Double, long: Double) {
        // reverse geocode coordinates into an address
        var geoCoder = Geocoder(this.context, Locale.getDefault())
        var address = geoCoder.getFromLocation(lat, long, 3)

        if (address.size > 0) {
            state = address[0].adminArea
            addressLine = address[0].getAddressLine(0)
            featureName = address[0].featureName
        }
    }
```

We create an instance of the [Geocoder](https://developer.android.com/reference/android/location/Geocoder) class in order to turn our latitude and longitude coordinates from the FusedLocationProviderClient into an actual address. Converting from coordinates to an address is _reverse geocoding_, while geocoding is the opposite.

We pass this function our latitude and longitude coordinates and call Geocoder's getFromLocation() function. You will notice that "address" is an array. The "3" parameter determines the maximum number of results that will be allowed into our array. This is because there may be several ways to express the location at a specific latitude and longitude. We arbitrarily set our maxResults to 3, but we only use the first result (address[0]). You can see that we chose to extract the location's admin area, address line, and feature name (which is only distinct from the address for certain locations -- you will notice that in our Berlin example screenshot, feature name is "Mitte", which means "Middle", or City Center).

There are a number of properties that can be accessed for this address. We used adminArea, addressLine, and featureName. Others we could have used are countryCode, countryName, locality, phone, and postalCode. Feel free to experiment with these and see which values you get.

As mentioned previously, this geocoder code will _not work_ without some kind of backend, as mentioned in [its official documentation](https://developer.android.com/reference/android/location/Geocoder). This is why we added Places to our dependencies in the Getting Started section. We cannot reverse geocode our coordinates if there is no location data to reverse geocode them into.

8) That is it for the core functionality of the app. For some extra functionality, let's give the user the option to send their location to the Google Maps app by pressing the "Open In Maps" button. Add this click listener to your onViewCreated():

```
openButton.setOnClickListener{ v ->
            if (latitude == 0.0 && longitude == 0.0) {
                addvalue.text = "Tap \"Update Location\" first!"
            } else {
                // creates an Intent that will load a map of the location
                val gmmIntentUri = Uri.parse("geo:$latitude,$longitude?q=${latitude},${longitude}(Currentparkingspot)")
                val mapIntent = Intent(Intent.ACTION_VIEW, gmmIntentUri)
                mapIntent.setPackage("com.google.android.apps.maps")
                startActivity(mapIntent)
            }
        }
```

First we check to see if the latitude and longitude coordinates haven't changed from their default 0.0 values (realistically it would be possible for someone to be at latitude 0.0 and longitude 0.0, but that is highly unlikely). If they are still their default values, we update the screen to let the user know to tap "Update Location" first so their location can be retrieved. 

Once we have a location to work with, we create an Intent that loads a map of the location, and call startActivity() on that intent. Note the part where we send our specific $latitude and $longitude values. Once pressing the "Open In Maps" button, the Google Maps app will open on the user's device with their location coordinates already loaded. 

There are a number of ways to implement Google Maps Intents for Android. You can read more about the process [here](https://developers.google.com/maps/documentation/urls/android-intents).

9) You're done! Now you're able to obtain a device's last known location and reverse geocode its coordinates into an address. Implement Android's Navigation Component and Recycler View to fully flesh out the application.

## Conclusion

This tutorial covers a common use case for our focus area, which is location retrieval. There are myriad other use cases and implementations which can be explored on the Google developer page for [Location and Context APIs](https://developers.google.com/location-context/). Another interesting area for further study is [Geofencing](https://developers.google.com/location-context/geofencing), which deals with defined parameters (_geofences_) surrounding areas of interest. You can also read more on what the [Places API](https://cloud.google.com/maps-platform/places/) has to offer apart from Geocoding support. It is often used in conjunction with Google Maps.

An alternative to the Play Services' Fused Location Provider is Android's Location API, which instead uses [LocationManager](https://developer.android.com/reference/android/location/LocationManager). This provides the same functionality with slightly different implementation. Google's Location Services provides better battery performance and higher accuracy. You can read more about the differences between these services and their implementations [here](https://stackoverflow.com/questions/33022662/android-locationmanager-vs-google-play-services).

When testing these features in an emulator, make sure to set the location of the emulator before running your app. This image shows how to do that:

![Image](https://i.imgur.com/QCTyUB9.png)


[Here](https://github.com/TrevorSpitzley/CIS357_FinalProject/tree/main) is the Github repository for our full app.


