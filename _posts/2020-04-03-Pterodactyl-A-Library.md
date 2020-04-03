---
title: "Pterodactyl: A Library to Automate Push Notification UI Testing"
header:
  og_image: /assets/images/2020-04-03-Pterodactyl-A-Library/pterodactyl.jpg
categories:
  - iOS
tags:
  - iOS
  - Testing
  - UI Testing
  - Push Notifications
---
 {% include captioned-image.html
    src="2020-04-03-Pterodactyl-A-Library/pterodactyl.jpg" caption="Image by  <a href=\"https://pixabay.com/users/elvina1332-7787848/\">Эльвина Якубова</a> from <a href=\"https://pixabay.com\">Pixabay</a>"
%}

One of the changes that came in Xcode 11.4 was the ability to send push notifications to the simulator. This is welcome addition to Xcode, since previously push notification testing was always limited to the device (as discussed in a [previous blog post of mine](https://mattstanford.dev/ios/push_notification-ui-testing/)). What was noticeably lacking (at least in my eyes) was a way to automate these through UI tests.

## Sending Push Notification to the Simulator in Xcode

Before getting into the library, it's important to understand how Xcode wants you to use this new functionality.

To start, you'll have to create a file that contains the APNS JSON payload. The APNS payload is slightly different than a normal payload - there are two extra parameters you need to specify. One is the bundle identifier of your app, and the other is the simulator ID you are sending it to. Here's an example below:

{% highlight json %}
{% raw %}
{
	"simulatorId": "2AE8B36E-473C-4C87-A417-88FDEF4B9280",
	"appBundleId": "com.myAppCompany.exampleApp",
    "aps": { 
        "alert": "here's a simple push notificaiton message",
        "badge": 42,
    }
}
{% endraw %}
{% endhighlight %}

After you create that, are two ways to send the push notification to your simulator:

1. Drag the JSON file into the simulator
2. Use the xcrun command line tool to send it. An example command would be something like the following:

 `xcrun simctl push 2AE8B36E-473C-4C87-A417-88FDEF4B9280 com.myAppCompany.exampleApp myJsonFile.json`

## Pterodactyl

Apple unfortunately did not leave us an API to use this new functionality within XCUITest. Pterodactyl works around this limitation by starting up a local macOS server on your computer. This server listens for a specific HTTP request, and when it gets it, runs the applicable xcrun command. The library you add to your app streamlines all messy stuff like creating an appropriate JSON file and sending the right network request to the local server.

Here's an example of it in action:

![](/assets/images/2020-04-03-Pterodactyl-A-Library/pterodactyl_example.gif)

For more details about configuring and using the library, check out its Github page here: [https://github.com/mattstanford/Pterodactyl](https://github.com/mattstanford/Pterodactyl).

If you’re interested in discussing it more with me, feel free to talk to me on Twitter: [@MattStanford3](https://twitter.com/MattStanford3).

Thanks for reading!


