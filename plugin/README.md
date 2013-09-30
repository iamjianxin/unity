# Chartboost Android and iOS Unity Plugin

Use the Chartboost plugin for Unity to add many features to your mobile games including displaying interstitials and more apps pages.

### Getting Started

After you have set up your app on the Chartboost web portal, you are ready to begin integrating Chartboost into your Unity project.

First, import all the provided files from the `Assets` folder into your Unity project.  If you are just building for iOS, you can skip the Android subdirectory, and vice versa.  The files will be organized into a few different directories:

 -  `/Assets/Plugins/Chartboost` : C# classes that bridge the gap between Unity and Java or Objective-C, allowing you to use Chartboost in your C# code
 -  `/Assets/Plugins/Chartboost/demo` : a example scene with classes that demonstrates a very simple usage of the Chartboost plugin
 -  `/Assets/Plugins/iOS` : the iOS Chartboost library and Objective-C code that helps to integrate it within Unity
 -  `/Assets/Plugins/Android` : the Android Chartboost library files and the template Android manifest
 -  `/Assets/Plugins/Android/res` : resources necessary for Chartboost to be included in the Android application - some important parameters are set here

##### Details for Android

Please note that Chartboost only supports Android 2.2 and higher.  When building for Android you must set the minimum Android version of your Unity project to at least 2.2.
 
You may already have an `/Assets/Plugins/Android/AndroidManifest.xml` file in your project. If so, you can skip importing that file, and instead just copy and paste the information from the `ChartboostAndroidManifest.xml` file included in this distribution into the relevant sections of your own `AndroidManifest.xml` file. You can look at the `AndroidManifest.xml` file from this plugin for an example of how it is done.  Make sure to read the **Conflicts With Other Android Plugins** section below for more details.

Before doing anything, you must modify the file `/Assets/Plugins/Android/res/values/strings.xml`.  This file contains your application's Chartboost ID and signature to prepare the library.  You can find these values on the Chartboost web portal. 

### Sample

If you want to jump straight into things, just examine the files in the `demo` folder mentioned previously.  Be sure to watch the log when playing with the demo scenes, as some of the buttons will not have obvious effects.

However, we suggest you read over the rest of this document.  There are a number of important concepts to understand in order to properly integrate the Chartboost plugin in your Unity app.
 
### Usage

##### Chartboost Setup

You must first call the `CBBinding.init()` method in order to be able to fully use the Chartboost plugin.  We recommend that you create a `MonoBehaviour` object that will live in your scene and manage all of the interaction with Chartboost.  You can place this `init()` call in the `OnEnable()` method of your `MonoBehaviour`.  While most of the features in this plugin are used identically between platforms, the `init()` method is actually slightly different. Also, keep in mind that your iOS and Android apps will have different app IDs and signatures.  Here is an example of how to call this method:

```c#
#if UNITY_ANDROID
	CBBinding.init();
#elif UNITY_IPHONE
	CBBinding.init("app_id", "app_signature");
#endif
```

##### Android: Back Button

In order for Chartboost to properly observe the Android back button, you must add the following code to a script of yours that is attached to an active `MonoBehaviour`.  `CBBinding.onBackPressed()` returns true when Chartboost has responded to the back button (for example, by closing an interstitial). You can replace `Application.Quit()` with anything you wish, as this will be the code executed when Chartboost does not respond to the back button.

```c#
public void Update() {
	if (Application.platform == RuntimePlatform.Android) {
		if (Input.GetKeyUp(KeyCode.Escape)) {
			if (CBBinding.onBackPressed())
				return;
			else
				Application.Quit();
		}
	}
}
```

##### Calling Chartboost methods

In `/Assets/Plugins/Chartboost/CBBinding.cs` you will find the Unity-to-native methods used to interact with the Chartboost plugin:

```c#
/// Initializes the Chartboost plugin and, on iOS, records the beginning of a user session
public static void init()

/// Caches an interstitial. Location is optional. Pass in null if you do not want to specify the location.
public static void cacheInterstitial( string location )

/// Checks to see if an interstitial is cached
public static bool hasCachedInterstitial( string location )

/// Shows an interstitial. Location is optional. Pass in null if you do not want to specify the location.
public static void showInterstitial( string location )

/// Caches the more apps screen
public static void cacheMoreApps()

/// Checks to see if the more apps screen is cached
public static bool hasCachedMoreApps()

/// Shows the more apps screen
public static void showMoreApps()

/// Returns true if an impression (interstitial or more apps page) is currently visible
public static bool isImpressionVisible()

/// Android only: return whether impressions are shown using an additional activity instead
///   of just overlaying on top of the Unity activity.  Default is true.
/// See `Plugins/Android/res/values/strings.xml` to set this value.
public static bool getImpressionsUseActivities()

/// Forces the orientation of impressions to the given orientation
public static void forceOrientation( ScreenOrientation orient )

/// Android only: used to notify Chartboost that the Android back button has been pressed
/// Returns true to indicate that Chartboost has handled the event and it should not be further processed
public static bool onBackPressed()

/// Tracks an event
public static void trackEvent( string eventIdentifier )

/// Tracks an event with additional metadata
public static void trackEventWithMetadata( string eventIdentifier, Hashtable metadata )

/// Tracks an event with a value
public static void trackEventWithValue( string eventIdentifier, float value )

/// Tracks an event with a value and additional metadata
public static void trackEventWithValueAndMetadata( string eventIdentifier, float value, Hashtable metadata )
```

##### Listening to Chartboost Events

Chartboost fires off many different events to inform you of the status of impressions.  In order to react these events, you must explicitly listen for them.  The best place to do this is the `OnEnable()` method of the Chartboost `MonoBehaviour` that you have created.  It is also good practice to stop listening to the events in `OnDisable()`.  Here is an example:

```c#
void OnEnable()
{
	// Listen to some interstitial-related events
	CBManager.didFailToLoadInterstitialEvent += didFailToLoadInterstitialEvent;
	CBManager.didCloseInterstitialEvent += didCloseInterstitialEvent;
	CBManager.didCacheInterstitialEvent += didCacheInterstitialEvent;
	CBManager.didShowInterstitialEvent += didShowInterstitialEvent;
}


void OnDisable()
{
	// Remove event handlers
	CBManager.didFailToLoadInterstitialEvent -= didFailToLoadInterstitialEvent;
	CBManager.didCloseInterstitialEvent -= didCloseInterstitialEvent;
	CBManager.didCacheInterstitialEvent -= didCacheInterstitialEvent;
	CBManager.didShowInterstitialEvent -= didShowInterstitialEvent;
}
```

On the left side of each line is the name of the Chartboost event.  All events will start with `CBManager`.  On the right side of each line is the name of the method you have created which will be informed of the event.  It doesn't have to have the same name as the event you are listening to, but there's probably no good reason to name it anything else.

In `/Assets/Plugins/Chartboost/CBManager.cs` you will find all of the events that are available to listen to:

```c#
/// Fired when an interstitial fails to load
/// First parameter is the location.
public static event Action<string> didFailToLoadInterstitialEvent;

/// Fired when an interstitial is finished via any method
/// This will always be paired with either a close or click event
/// First parameter is the location.
public static event Action<string> didDismissInterstitialEvent;

/// Fired when an interstitial is closed (i.e. by tapping the X or hitting the Android back button)
/// First parameter is the location.
public static event Action<string> didCloseInterstitialEvent;

/// Fired when an interstitial is clicked
/// First parameter is the location.
public static event Action<string> didClickInterstitialEvent;

/// Fired when an interstitial is cached
/// First parameter is the location.
public static event Action<string> didCacheInterstitialEvent;

/// Fired when an interstitial is shown
/// First parameter is the location.
public static event Action<string> didShowInterstitialEvent;

/// Fired when the more apps screen fails to load
public static event Action didFailToLoadMoreAppsEvent;

/// Fired when the more apps screen is finished via any method
/// This will always be paired with either a close or click event
public static event Action didDismissMoreAppsEvent;

/// Fired when the more apps screen is closed (i.e. by tapping the X or hitting the Android back button)
public static event Action didCloseMoreAppsEvent;

/// Fired when a listing on the more apps screen is clicked
public static event Action didClickMoreAppsEvent;

/// Fired when the more apps screen is cached
public static event Action didCacheMoreAppsEvent;

/// Fired when the more app screen is shown
public static event Action didShowMoreAppsEvent;
```

##### Android: Customizing Pause Behavior During Impressions

Normally in Android, when Chartboost impressions are shown, your Unity app temporarily halts, and due to limitations in Unity, it turns black while impressions are animating in and out.  This may be undesirable, but with a bit of customization, it can be fixed using the following procedure:

1. Navigate to the `/Assets/Plugins/Android/res/values/strings.xml` file and set the value of the `chartboost_unity_impressions_use_activities` entry to `false`.
2. Now, Unity keeps running in the background while impressions are shown.  You'll probably want your game to be paused, and we've provided a default implementation of that behavior by setting Unity's `Time.timeScale` to `0`.  You can change this by editing the code in `CBManager.doCustomPause(boolean)`.
3. There's another problem -- due to Unity optimizations used in Android 2.3+, you might notice that touch input not only affects the Chartboost impressions, but also passes right through to your Unity app!  To fix this, you'll have to check if a Chartboost impression is visible using the `CBBinding.isImpressionVisible()` before reacting to touch input.  For example, if you are using Unity's built-in GUI system, you can add the following code before drawing any controls in the `OnGUI()` method:

```c#
// disable GUI controls if a Chartboost impression is visible
GUI.enabled = !CBBinding.isImpressionVisible();
```

### Android: Conflicts with Other Plugins

If you use other plugins, it is possible that you may need to do some manual changes to allow both plugins to work together.

- Does each plugin come with a `/Assets/Plugins/Android/AndroidManifest.xml` file?  If so, you will need to ensure that the file you use in your project includes the items from each plugin's manifest file.  If you don't want to use the `AndroidManifest.xml` file from our plugin, as an alternative we have provided a file `ChartboostAndroidManifest.xml` that includes the information you will need to copy into the `AndroidManifest.xml` file with which you build your project.

- The Chartboost plugin uses its own customized activities to work properly.  If another plugin you use does this as well, you will need to export to an external IDE such as Eclipse or Android Studio when building and follow some more complex instructions, outlined below in the **Exporting to Android Project** section.  You can tell if a plugin uses custom activities by looking in its manifest file for any `<activity>` tag whose name does not start with `com.unity3d.player` and includes the following XML inside of it:

```xml
<intent-filter>
	<action android:name="android.intent.action.MAIN" />
	<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```
 
### Finishing Up

##### Building for iOS

The first time you build your project, you must do a *Build* (not a *Build and Run*) so that everything is set up properly.  Once it finishes, load up the Xcode project that is created and make the following modification:

- Navigate to the *Build Phases* settings tab for your target, and make sure that you are linking against the `AdSupport` framework.  You can weak-link `AdSupport` by selecting "Optional" next to it.

In the future, you can select the *Build and Run* method, as long use you use the *Append* option rather than *Replace* so that your changes are kept around.

##### Building for Android

To build your project, build it directly to an APK file, or export to an Android Project if you would like to make further modifications in Java (see the next section for details).

### Exporting to Android Project

If you plan to export your Unity project for Android to make modifications before it is compiled, there are some special considerations you must take into account.

###### Obfuscation

If you would like to obfuscate your application using Proguard, make sure to add the following line to your Proguard configuration file:

	-keep class com.chartboost.sdk.** { *; }

###### Customized Activities

If you want to make customizations to the activity classes that you use, then (1) make the `UnityPlayerActivity` and `UnityPlayerNativeActivity` classes that Unity made for you in your Android project extend from `CBUnityPlayerActivity` and `CBUnityPlayerNativeActivity` instead, and (2) remove ALL of the code Unity put in the two classes for you (you can leave `UnityPlayerProxyActivity` alone).  Alternatively, if you need to have your activities extend from other base activity classes (if you are using another plugin that requires it, for example), then incorporate the following code into your regular and native activity classes:

```java
@Override
public void onCreate(Bundle b) {
	super.onCreate(b);
    CBPlugin.onCreate(this);
}

@Override
public void onStart() {
	super.onStart();
	Chartboost.sharedChartboost().onStart(this);
	Chartboost.sharedChartboost().startSession();
}

@Override
public void onStop() {
	Chartboost.sharedChartboost().onStop(this);
	super.onStop();
}

@Override
public void onDestroy() {
	Chartboost.sharedChartboost().onDestroy(this);
	super.onDestroy();
}
```
	
If you already override some of these lifecycle methods, then you will need to merge your code with the code above manually.

Additionally, make sure you check what activities are listed in your `AndroidManifest.xml` file. If you are using modifications of the activities that Unity created for you, change the three Android activity names like `com.chartboost.sdk.unity.CBUnityPlayerProxyActivity` to their counterparts in your project, which you can format like `.UnityPlayerProxyActivity`, `.UnityPlayerActivity`, and `.UnityPlayerNativeActivity`.