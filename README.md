# OculusMobileSDKHeadTracking

The Oculus Mobile SDK allows to access a highly accurate head tracking hardware system available on devices like the Samsung Gear VR by Samsung and Oculus. The purpose of this library is to be able to expose only the head tracking mechanism from the Oculus Mobile SDK to a C++ or a java API. Any application or library can use this information to develop products that can take advantage of the high accuracy information that is provided by these sensors.

This project is the base for a bigger goal: provide Oculus Mobile Head Tracking capabilities to web applications. Other projects help achieve this goal, so if you are interested in WebVR, please review the following git repos:

* [OculusMobileSDKHeadTrackingXWalkViewExtension](https://github.com/judax/OculusMobileSDKHeadTrackingXWalkViewExtension): A Crosswalk extension to expose the Oculus Mobile SDK Head Tracking in a JavaScript/browser based environment based on the Crosswalk webview. 

  **IMPORTANT NOTE:** If you want to test WebVR on a Gear VR using its accurate head tracking in action right now, check the [How to use the library](https://github.com/judax/OculusMobileSDKHeadTrackingXWalkViewExtension#how-to-use-the-library) section of the OculusMobileSDKHeadTrackingXWalkViewExtension project.

* [OculusMobileSDKHeadTrackingCordovaPlugin](https://github.com/judax/cordova-plugin-oculusmobilesdkheadtracking.git): A Cordova plugin to expose the Oculus Mobile SDK Head Tracking in a JavaScript/browser based environment (under development, not fully functional yet).
* [OculusMobileSDKHeadTrackingWebVR](https://github.com/judax/OculusMobileSDKHeadTrackingWebVR): A JavaScript file that injects the WebVR API using the underlying Oculus Mobile SDK Head Tracking mechanism exposed to JS through the Crosswalk extension or the Cordova plugin.

## Folder Structure

* **3rdparty**: The third party libraries used to build this library. In this case the Oculus Mobile SDK libraries are needed.
* **build**: The final build of this library. You can use these final products if you do not want to build the library yourself.
* **java**: The java side of the library. This is the final API that other projects will see/use.
* **javadoc**: The javadoc of the java side of the library.
* **jni**: The C++ side of the library. In fact, this code is the one that handles the Oculus Mobile SDK communication. All the makefiles to be able to build the code are provided.
* **test**: A simple test that shows how to use the library. The test shows head tracking values on screen.

## Assumptions

* This documentation will assume that you have experience on Android development and more specifically on using Eclipse to develop Android projects.
* This project provides some already built libraries. Of course, you can decide to use other versions of the same libraries or even build them yourself.
* Although some links and information will be provided, some knowledge on Oculus Mobile app development (specifically on the Samsung Gear VR) might come handy.
* You will need to have the Android SDK installed along with Eclipse and the ADT plugin for Eclipse. If you want to build the Android native source code too, you will need to have at least the Android NDK version r9b or above installed.

## Setup your Samsung/Oculus Gear VR

If you already know how to setup your Samsung Gear VR compatible device to install and execute custom VR apps, you can skip this section and jump directly to [How to use the library](https://github.com/judax/OculusMobileSDKHeadTracking#how-to-use-the-library).

By default, Samsung devices that are compatible with the Gear VR (Note 4 and Galaxy S6 for now) come with some Oculus/Gear VR services preinstalled. These services make developing and installing custom apps quite difficult so setting up your device to act as a developer is important among other steps. There are seveal tutorials online but the infomation is quite scattered so let's review the most important steps to setup your device and get it ready for custom app installation/execution:

1. **Activate Gear VR developer mode:** Similarly to what happens when [activating the developer mode on Android](http://developer.android.com/tools/device.html), you can activate a developer mode for the Gear VR. In order to do so, on your Samsung Android device, go to `Settings->Applications->Application manager` and look for the `Gear VR Service`. 

  ![](markdown/images/GearVRDevMode01.jpg "")

  Inside it your will see a button that states `MANAGE STORAGE` and once you press on it a field labeled `VR Service Version` show on screen. As it happens to enable the Android developer mode, you need to tap 7 times repeatedly over this field and you will see how the `Developer mode` is activated. You can enable or disable this mode at anytime from this option.

  <table>
  	<tr>
  		<td style="vertical-align:top;">
  			<img src="markdown/images/GearVRDevMode02.jpg"/>
  		</td>
  		<td style="vertical-align:top;">
  			<img src="markdown/images/GearVRDevMode03.jpg"/>
  		</td>
  	</tr>
  </table>

2. **Disable the Oculus Home app:** 

  **UPDATE: This step is no longer needed.** It was brought to my attention (thanks to Michael Blix from Samsung) that there is the option to specify that the Android app can be executed without having to launch the Oculus Home by changing the manifest file. All the information on how to do it for Unity (applicable for your own Android project) is [here](https://developer.oculus.com/documentation/publish/latest/concepts/publish-mobile-manifest/). In the end, everything comes down to adding the following tag to your Android Application tag.

  ```
  <meta-data android:name="com.samsung.android.vr.application.mode" android:value="vr_only"/>
  ```

  The manifest files of the tests have been updated in these repos. Anyway, in case you also want to disable the Oculus Home for whatever reason, I will keep the explanation on how to disable the Oculus Home service.

  Every time you connect your Samsung Android device to the Samsung Gear VR USB connector, the Oculus Home application is launched. It is not mandatory to deactivate this behaviour as you can easily kill the app once is launched, but when your are developing and testing a Oculus Mobile app, it is preferrable not to have it taking over every time disconnect your device from the USB to the computer and connect it to the headset to test your progress. This is specially important with the Note 4 based first version of the Gear VR as killing the Oculus Home app is trickier than in the Galaxy S6 model. The best (but not free) way to disable the Oculus Home app is to install the [Package Disabler Pro app from GooglePlay](https://play.google.com/store/apps/details?id=com.ospolice.packagedisablerpro&hl=en). Once downloaded and installed if you open the Package Disabler Pro, it will show all the services that are running on your device. The fastest way to find the Oculus Home service is to use the filter text field on the top of the screen inside the Package Disabler Pro. Just type Oculus and it should filter all the apps and how the one that is needed to be disabled. Then simply select the checkbox on the right side of the app name and the Package Disabler Pro will handle it. Any disabled app can be re-enabled anytime by just unselecting the checkbox, so anytime you feel like having the oculus home being launched once you connect your device to the headset, you can have this behaviour back in no time.

  It seems that some developers using the Gear VR are disabling the "Gear VR Service" instead of just the "Oculus Home" app. The Oculus Mobile SDK requires that the "Gear VR Service" is up and running or the app might crash with a strange NumberFormatException error. Please, ensure that you have the Gear VR Service up and working (checking on the Application Manager list for example).

3. **Get the Oculus Signature File (OSIG) for your device:** The Oculus Mobile SDK requires your app to be "signed" in order to be launched. This signing process forbids your create an app and distribute it to any device freely :(. It is also prettry simple: Just create the Oculus Signature ID file (OSIG) for your device and copy it to the assets folder of the device. In order to create the OSIG file you will need your Android device id. Once you know your device id, you can introduce it in the [OSIG generator tool by Oculus](https://developer.oculus.com/osig/).

## How to use the library

### Use the `test` project

The easiest way to have a glimpse on how to use the library is to check the `test` project. The project is ready to be executed, so just import it to Eclipse. The only missing piece is that in order to work on a Samsung Device you will need to generate your OSIG file and copy it to the `assets` folder of the `test` project. Check the [instructions above](https://github.com/judax/OculusMobileSDKHeadTracking#setup-your-samsungoculus-gear-vr).

### Use the library in your own project

You might as well want to create your own project that uses the libraries. The final products are stored in the `build` folder. Remember that this library is also dependant on the Oculus Mobile SDK which final products are also provided along in this repository for your convenience. These are the steps you need to follow in order to use this library:

1. The OculusMobileSDKHeadTracking final libraries are available at the `build` folder. Just copy all the content of this `build` folder to the `libs` folder of your Eclipse project.
2. This project depends on the Oculus Mobile SDK. Luckily the necessary libraries are provided the `3rdparty` folder. These are the files you need to copy to the `libs` folder of your Eclipse project:
  * **`3rdparty/ovr_sdk_mobile_1.0.0.0/SystemUtils.jar`**
  * **`3rdparty/ovr_sdk_mobile_1.0.0.0/VrApi.jar`**

  **NOTE:** The `libvrapi.so` file is directly provided along with the `liboculusmobilesdkheadtracking.so` file. A copy of the same version of the file is also included in the `3rdparty/ovr_sdk_mobile_1.0.0.0/armeabi-v7a` folder in case you need it.
3. Use the library inside your code.
  1. Create an `OculusMobileSDKHeadTracking` instance.
  2. Implement the `OculusMobileSDKHeadTrackingListener` interface. This step is not mandatory but listening to the events can come handy to know when the tracking has started or to be notified about possible errors. You can acquire the head tracking values by calling the `getData` method.
  3. Call the `start` method of the instance passing a reference to the Activity that is using the library. This call should most likely be made from the `onCreate` of the activity that has instanitated the `OculusMobileSDKHeadTracking` class.
  4. Call the `getView` method of the instance to get a view that needs to be added somehow in the view hierarchy of your app.
  5. Call the `resume`, `pause` and `stop` methods of the instance in the corresponding `onResume`, `onPause` and `onDestroy` of the Activity.

  Here is some source code of a possible Activity skeleton that uses the library:

  ```
  package com.judax.oculusmobilesdkheadtracking.test;
  
  import com.judax.oculusmobilesdkheadtracking.OculusMobileSDKHeadTracking;
  import com.judax.oculusmobilesdkheadtracking.OculusMobileSDKHeadTrackingData;
  import com.judax.oculusmobilesdkheadtracking.OculusMobileSDKHeadTrackingListener;
  
  import android.app.Activity;
  import android.os.Bundle;
  import android.widget.FrameLayout;
  
  public class OculusMobileSDKHeadTrackingTestActivity extends Activity
  {
	  private OculusMobileSDKHeadTracking oculusMobileSDKHeadTracking = new OculusMobileSDKHeadTracking();
  	
  	  @Override
    	protected void onCreate(Bundle savedInstanceState)
    	{
  		super.onCreate(savedInstanceState);
  		
  	    FrameLayout layout = new FrameLayout(this);
  
  		// Initialize the oculus mobile sdk head tracking
  		oculusMobileSDKHeadTracking.start(this);
  
  		// Listen to the Oculus mobile sdk head tracking events
  		oculusMobileSDKHeadTracking.addOculusMobileSDKHeadTrackingListener(new OculusMobileSDKHeadTrackingListener()
  		{
  			@Override
  			public void headTrackingStarted(OculusMobileSDKHeadTracking oculusMobileSDKHeadTracking, OculusMobileSDKHeadTrackingData data)
  			{
  			}
  			
  			@Override
  			public void headTrackingError(OculusMobileSDKHeadTracking oculusMobileSDKHeadTracking, final String errorMessage)
  			{
  			}
  		});
  
  		// Add the view provided by the oculus mobile head tracking instance to the view hierarchy
  	    layout.addView(oculusMobileSDKHeadTracking.getView());
  
  	    // ...
  	    // Add your layout/UI here to the main layout. As the main layout is a framelayout, your UI will be on top of the view provided by the OculusMobileSDKHeadTracking that will be occluded.
  	    // ....
  	    
  	    setContentView(layout);
  	}
  	
  	@Override
  	protected void onResume()
  	{
  		super.onResume();
  		oculusMobileSDKHeadTracking.resume();
  	}
  	
  	@Override
  	protected void onPause()
  	{
  		super.onPause();
  		oculusMobileSDKHeadTracking.pause();
  	}
  	
  	@Override
  	protected void onDestroy()
  	{
  		super.onDestroy();
  		oculusMobileSDKHeadTracking.stop();
  	}
  }
  ```

4. Include your Oculus Signature File (OSIG) in the assets folder of your project. You can generate yout Oculus OSIG file at [https://developer.oculus.com/osig/](https://developer.oculus.com/osig/). Check the section above on how to setup your Samsung Device for Gear VR development if necessary.

## How to build the library

If you would like to contribute to this repo or clone/fork it and modify it, you might want to know how to build the libraries yourself. There are two main elements for the library:

1. **The C++ code:** An Android native library that uses the Oculus Mobile SDK. It generates a final dynamic library (.so) file: `build/armeabi-v7a/libOculusMobileSDKHeadTracking.so`. All the code for this library is available in the `jni` folder along with the Android NDK makefiles. All the necessary dependencies are also provided along with the repo as mentioned in the section below. There is also an optional bash script called `BuildAndCopyToTest.sh` inside the `jni` folder that executes the ndk-build command. I highly recommend to at least check this script out to see how the references to the makefiles are provided along with the final destination folder (`build`) to create the final libraries and the intermediate obj files. This script also makes a copy of the final library to the `test` project so it is always up to date with the modifications made to the native code. In order to be able to use the script, the `ANDROID_NDK_PATH` environment variable needs to be defined pointing to the folder where the Android NDK is installed in your system. All the native C++ code is based on the Oculus Mobile SDK VRSample project [VrCubeWorld_SurfaceView](https://developer.oculus.com/documentation/mobilesdk/latest/concepts/mobile-native-samples/) but everything that was not needed has been stripped out and a main class has been implemented.

2. **The Java code:** An Android Java library that establishes both a link to the native C++ code and exposes a Java API so the library can be used by other projects. It generates a .jar file: `build/oculusmobilesdkheadtracking.jar`. The code of the library is in the `java` folder where a Eclipse project is waiting for you to import it to your workspace. This library handles both the loading of the native library (.so) and handles all the call from and to native code. It also exposes the API other Android projects can use to have access to the head tracking. Check the `javadoc` folder for full documentation on the public API. In order to generate the javadoc, remember to set the `ANDROID_HOME` environment variable. All the necessary libraries are already available in the libs folder of the project. The project also includes Ant Builders to copy the final jar product file and to generate the javadoc documentation.

### Provided Third Party Libraries

This project provides all the necessary libraries to be able to build the final products/library inside the `3rdparty` folder. The minimum elements (headers, .a-s, .so-s, .jar-s, ...) from the Oculus Mobile SDK are included with no modification from the original Oculus Mobile SDK. The `3rdparty/ovr_sdk_mobile_1.0.0.0/armeabi-v7a/libopenglloader.a` library is the only one that the Oculus Mobile SDK does not directly provide in a binary form. I have simply built it using the provided Android.mk file in the Oculus Mobile SDK and copied the final .a result. The full Oculus Mobile SDK can be downloaded [here](https://developer.oculus.com/downloads/) if you would like to use a different version or even have the full library. Of course, some modifications might be needed in order to use a different version as the API may have changed and most likely a new build of the openglloader library might be needed.

### WTF, an XCode project for Android development?

I like to use IDEs, I think they provide a great way to work over code, check on compilation errors, etc. On mobile, I develop for both iOS and Android and both in C++ (with Objective-C and Java) and having the same IDE comes handy. XCode allows to call external scripts (ndk-build) and the output is also highlighted over the code in screen, making C++ JNI development quite simple. Is not perfect, as it does not provide debugging or code syndication but it certainly helps. Anyway, using XCode is completely optional as in the end, the ndk-build command, the Android.mk and Application.mk are all that is needed to compile the Android native side of the library. The XCode project simply uses the `jni/BuildAndCopyToTest.sh` underneath as all the default build phases have been removed.

If you would like to use a different IDE, feel free to fork the project and provide a pull request with an Android Studio project for example.

### Used System/Software

To build and test this project, this is all the information about the system I have used and the software involved:

* A MacBookPro 13 inches with El Capitan Mac OSX version.
* Eclipse Indigo
* Android SDK for Mac
* Android NDK r9b
* XCode 7.2
* Samsung Galaxy S6 with Android 5.0.2
* Samsung Gear VR

**NOTE:** As some of the Eclipse projects include Builders that execute ant tasks.

## Related Projects

* [OculusMobileSDKHeadTrackingXWalkViewExtension](https://github.com/judax/OculusMobileSDKHeadTrackingXWalkViewExtension): A Crosswalk extension to expose the Oculus Mobile SDK Head Tracking in a JavaScript/browser based environment based on the Crosswalk webview. 
* [OculusMobileSDKHeadTrackingCordovaPlugin](https://github.com/judax/cordova-plugin-oculusmobilesdkheadtracking.git): A Cordova plugin to expose the Oculus Mobile SDK Head Tracking in a JavaScript/browser based environment (under development, not fully functional yet).
* [OculusMobileSDKHeadTrackingWebVR](https://github.com/judax/OculusMobileSDKHeadTrackingWebVR): A JavaScript file that injects the WebVR API using the underlying Oculus Mobile SDK Head Tracking mechanism exposed to JS through the Crosswalk extension or the Cordova plugin.

## Other References

Oculus Mobile SDK website

## Future work, improvements and random ideas

* Improve the C++ code to decouple it from the Java code in case someone wants to use it from C++ directly. It might be complicated as, at least for now, an Activity and a surfaceview are needed, but there is room for improvement for sure. This improvement could lead to a better integration with already c++ code on other projects like Chromium.
* Try to contact Oculus/Samsung to see if there is a better way/SDK to just access the Gear VR gyro driver without having to do so much Java/C++ code (create a surfaceview, a GL context, ...).
* Add an Android Studio based project solution too and provide the library as an Android AAR.
* Expose the proximity sensor functionality
