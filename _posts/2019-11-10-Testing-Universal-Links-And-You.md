---
title: "Testing Universal Links in iOS & You"
header:
  og_image: /assets/images/2019-11-10-Testing-Universal-Links-And-You/ghost-hand.jpg
categories:
  - iOS
tags:
  - iOS
  - Testing
  - UI Testing
  - Universal Links
  - Custom Schemes
---
 {% include captioned-image.html
    src="2019-11-10-Testing-Universal-Links-And-You/ghost-hand.jpg" caption="Image by  <a href=\"https://pixabay.com/users/alexas_fotos-686414\">Alexas_Fotos</a> from <a href=\"https://pixabay.com\">Pixabay</a>"
%}

Universal links and custom schemes are one of those things that is always more complicated than you’ve originally thought. It’s just clicking a link to get into the app, right? Later on, however, you realize that the app could be in a myriad of states before a user taps it. What if the user is logged out? What if they're viewing a modal 2 screens deep in the settings view?

Sounds like a good thing to have automated testing for! Unfortunately UI testing for universal links and custom schemes has always been a little perilous.

## **The current state of UI Testing custom schemes and universal links**

Currently, the fastest way to test custom schemes is to use the Safari app that is built into the simulator. Basically you simulate tapping the home button, launch the Safari app, and simulate typing the URL into the address bar. Something like this:

{% highlight swift %}
{% raw %}

 func testSafari() {
        // Launch safari
        let safari = XCUIApplication(bundleIdentifier: "com.apple.mobilesafari")
        safari.launch()
        
        // Make sure Safari’s URL bar has appeared
        let safariUrlButtonExists = safari.buttons["URL"].waitForExistence(timeout: 5)
        XCTAssert(safariUrlButtonExists)
        
        // Type in the text of the URL
        safari.typeText("testCoco://TEST\n")
        
        // Respond to the modal asking you if it's ok to launch the app
        safari.buttons["Open"].tap()
        
        // Wait for your application to enter the foreground
        _ = app.wait(for: .runningForeground, timeout: 5)
    }

{% endraw %}
{% endhighlight %}

However, using this method will NOT work if you use a universal link. For some reason, iOS only opens a universal link if it is “tapped on”. An alternative approach to testing this was written about in a post on Branch.io’s blog here: [https://blog.branch.io/ui-testing-universal-links-in-xcode-9/](https://blog.branch.io/ui-testing-universal-links-in-xcode-9/). Here’s a basic rundown of how a UI test using this technique works:
* Put your app in the background with the home button
* Launch the Messages app
* Tap one of the always-present conversations the simulator has (either “Kate Bell” or “John Appleseed”)
* Type in your URL in message box
* Once your url appears in the conversation, tap it
* Your app should launch

Here's what it looks like in practice (minus it actually opening the app, I didn't configure a universal link for this blog post, sorry):

![](/assets/images/2019-11-10-Testing-Universal-Links-And-You/messages_test.gif)

## **Problems with using Messages**

I personally have been using the Messages approach to UI Testing universal links for a little while now. While the approach the above blog post worked pretty well at first, updates to the Simulator app have continually broken the process. Some things that have thrown me curveballs were:
* Messages now shows a “What’s New in Messages” intro modal view on first run. This happens more often than you might think, for some reason cloned simulators (used when running tests in parallel mode) get started, or if you happen to reset your simulators. This modal must be dismissed IF it’s shown.
* Messages now also sometimes throws up the “new message” view on startup. I have not yet been able to determine why and how this happens. If it does happen, it needs to be dismissed so we our test can navigate the main table view of messages.
* Sending a message with no other text but the URL will cause Messages to just remove the message after a second or two. I’m not sure of the reason of this either, but the solution is send a link with some placeholder text in front of it (like this “here’s a link: https://my.universallink.com”)
* They are SLOW. Opening messages, navigating the message list, typing in the URL, takes a good amount of time (even compared to other UI tests).

So as you can see, dealing with messages is kind of a pain. With every update to Xcode, you may have to go through and fix all the things that broke with your universal link tests.

After a while of dealing with constant maintenance and slowness of these tests, I started to wonder if this was really the best way to go about testing these use cases. It seems like the Messages app is not really intended for these kinds of tests - I would wager it is present for the purposes of testing iMessage extensions or Sticker Packs.

## **So I Wrote a Library**

I started to think, what would be the IDEAL way to test these? It would probably involve an app on the simulator/phone that was actually designed to test universal links/custom schemes. Also, it should require the least amount of user interaction possible, so the tests could finish as fast as possible.

So I wrote an app that does exactly that. This app, which I named “GhostHand”, can be passed a URL in its launch parameters. Once it’s launched, it will immediately launch said URL in the "application(didFinishLaunchingWithOptions)” method. This will work with both custom schemes and universal links.

The tricky part of this was getting this into a library that could be distributed via CocoaPods or Carthage. In the end I got everything to work. The GhostHand app is pre-built and embedded into a framework called “GhostHandLib”. Any implementor of this library must make sure to run a simple run script in their UI Test target to trigger the install of the app on the simulator they are running the test against.

Here’s how the app works in practice, assuming you have it configured in your project correctly:

{% highlight swift %}
{% raw %}

 func testSample() {
        app.launch()
        let appBooted = app.staticTexts["Sample App for 👻"].waitForExistence(timeout: 5)
        XCTAssert(appBooted)
        
        //Tap the home button to put the app under test in the background
        XCUIDevice.shared.press(XCUIDevice.Button.home)
        
        //Use the GhostHand companion app to launch the custom scheme
        GhostHand.launch(url: "ghostHandSampleApp://TEST")
        
        //The app should be called by the GhostHand companion app
        let appShowedAgain = app.staticTexts["Sample App for 👻"].waitForExistence(timeout: 5)
        XCTAssert(appShowedAgain)
    }

{% endraw %}
{% endhighlight %}

And here’s what it looks like in action:

![](/assets/images/2019-11-10-Testing-Universal-Links-And-You/ghosthand_test.gif)

If you’re interested in using it yourself, here is the GitHub page: [https://github.com/mattstanford/GhostHand](https://github.com/mattstanford/GhostHand).

If you’re interested in discussing it more with me, feel free to talk to me on Twitter: [@MattStanford3](https://twitter.com/MattStanford3).
