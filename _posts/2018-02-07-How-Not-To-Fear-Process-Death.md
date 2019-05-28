---
title: "How Not to Fear Process Death"
header:
  image: /assets/images/2018-02-07-How-Not-To-Fear-Process-Death/death_title.jpeg
categories:
  - Android
tags:
  - Android
  - Process Death
---

![](/assets/images/2018-02-07-How-Not-To-Fear-Process-Death/death_title.jpeg)

If you’ve been writing Android apps for a while, you may have experienced what is known as “process death”. Process death will happen when the Android system decides your backgrounded app needs to be killed from memory to make room for other apps.

When your app is resumed after a process death, the activity that the user was last using will be recreated and will call its “onCreate” method. This might sound similar to a configuration change (for example, when the user rotates their phone), however additional data will have been lost on a process death such as the following:

* Singleton data

* Mutable static variables

* Data being stored in a retained fragment as described here: [https://developer.android.com/guide/topics/resources/runtime-changes.html#RetainingAnObject](https://developer.android.com/guide/topics/resources/runtime-changes.html#RetainingAnObject)

That last point deserves a bit of clarification. If you are currently saving off data in a retained fragment, in a process death, the fragment will actually still be returned by the FragmentManager, but its retained data will not!

So how can you mitigate this? There are a few ways:

* Persist any important data (like auth tokens) in SharedPreferences or a SQLite database. Re-query this data in your “OnCreate” method.

* Remember that the “SavedInstanceState” bundle **will** be saved. So one strategy would be to save off any Sington-esque data your app uses. You may have to do something like make a base activity class to accomplish this. Relying on all your activities to inherit from a base class could cause complications though.

* For the truly desperate, you could do a check for a process death in the “OnCreate()” method of all your activities, and then re-launch your launcher activity.

### Simulating Death

So that’s all well and good, but how do you test a process death? There are few different methods you can use to do this:.

* In the Logcat menu of Android Studio, there is a button with a red circle and an “x” in the middle of it (it might be in the overflow menu). This button is called “terminate process”. Simply put your app in the background, then press this button. 

![](/assets/images/2018-02-07-How-Not-To-Fear-Process-Death/death_1.png)

Behold! You have become process death.

I’ve found this doesn’t always work however, as Android will fully terminate your app and re-start it from the launch activity.

* Another way I’ve found to be fairly reliable is to run the following adb command in your terminal:

{% highlight shell %}
adb shell am kill <your package name>
{% endhighlight %}


### Debugging

Unfortunately, triggering a process death will also kill any debugger connection you might have running with the app. However, you can mitigate this as well! Perform the following steps to debug your app after a process death:

* Put the following line of code in the method you want to set a breakpoint in:

{% highlight java %}
    android.os.Debug.waitForDebugger();
{% endhighlight %}


* Run your app

* Go to the activity in question, trigger a process death

* You will note that Android Studio has disconnected the debugger (if you ran it in Debug mode)

* Restart the app by selecting it in the task switcher or on the launchpad

* You might notice the app freezes — this is because it is sitting on the line you put in above, waiting for you to attach a debugger

* Attach a debugger (In Android Studio, go to Run->Attach Debugger to Android Process)

Hopefully you now no longer are living in fear of process death! Your users will thank you (I’m sure).
