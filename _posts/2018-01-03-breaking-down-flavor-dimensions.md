---
title: "Breaking Down Android Flavor Dimensions"
canonical_url: "https://medium.com/@mds6058/localization-in-ios-and-how-to-make-it-not-suck-3adcbc3ec08f"
header:
  og_image: /assets/images/2018-01-03-breaking-down-flavor-dimensions/flavors.jpeg
categories:
  - Android
tags:
  - Android
  - Flavors
  - Flavor Dimensions
---

![](/assets/images/2018-01-03-breaking-down-flavor-dimensions/flavors.jpeg)

If you’ve recently updated to Android Studio 3.0, you may have received the following error message when making a gradle build:

“Error:All flavors must now belong to a named flavor dimension”

The error message also supplies a link with more information on the main Android docs site, however I was still a little confused after reading it.

In a nutshell, flavor dimensions allow you to separate out different groups of build settings into “dimensions”. Gradle will then combine then create build variants for every possible combination of each dimension.

But let’s break that down a little.

Say you have an app, and you want to have two separate build types, one for a free demo version, and the other for a paid, or “full” version. What’s one way to accomplish that? Build flavors!

You might create a section in your app’s build.gradle file like this:

{% highlight groovy %}
{% raw %}
    productFlavors {
     
       demo {
            buildConfigField "Boolean", "isDemo", "true"
        }
        full {
            buildConfigField "Boolean", "isDemo", "false"
        }
    }
{% endraw %}
{% endhighlight %}

With these flavors, you would most likely generate 4 different build configurations: demoDebug, demoRelease, fullDebug, and fullRelease.

But say your app connects to the network somehow, and you have a development and production server. You would like to be able to configure the server URL on a per-build basis. You could do that with a build config field as well! However, since we already have a build settings for the free/full build type, we have to duplicate a few things in the gradle file:

{% highlight groovy %}
{% raw %}
    productFlavors {
       
        demoDev {
            buildConfigField "Boolean", "isDemo", "true"
            buildConfigField "String", "serverUrl", '"http://dev.my-server.com"'
        }
        
        demoProd {
            buildConfigField "Boolean", "isDemo", "true"
            buildConfigField "String", "serverUrl", '"http://prod.my-server.com"'
        }
        
        fullDev {
            buildConfigField "Boolean", "isDemo", "false"
            buildConfigField "String", "serverUrl", '"http://dev.my-server.com"'        }
        
        fullProd {
            buildConfigField "Boolean", "isDemo", "false"
            buildConfigField "String", "serverUrl", '"http://prod.my-server.com"'
        }
    }
{% endraw %}
{% endhighlight %}

OK, fairly easy to understand, but do you see a problem? We seem to be duplicating a lot of information here. If you were to add any **other** settings, you can see how this could quickly get out of hand.

Flavor dimensions can help here to eliminate this redundancy. The above can be replaced by the following gradle code:

{% highlight groovy %}
{% raw %}
    flavorDimensions "tier", "serverUrl"

    productFlavors {
        demo {
            dimension "tier"
            buildConfigField "Boolean", "isDemo", "true"
        }
        full {
            dimension "tier"
            buildConfigField "Boolean", "isDemo", "false"
        }
        
        dev {
            dimension "serverUrl"
            buildConfigField "String", "serverUrl", '"http://dev.my-server.com"'
        }
    
        prod {
            dimension "serverUrl"
            buildConfigField "String", "serverUrl", '"http://prod.my-server.com"'
        }
    }
{% endraw %}
{% endhighlight %}

This will create the 8 different build configurations for every possible combination of these “flavor dimensions: demoDevDebug, demoDevRelease, demoProdDebug, etc.

So hopefully that sheds a little light on this topic. This can be pretty handy if you have a non-trivial build configuration. Of course you can find more information on these in the official Android documentation here: [https://developer.android.com/studio/build/build-variants.html#flavor-dimensions](https://developer.android.com/studio/build/build-variants.html#flavor-dimensions) .

That’s all for now. This is my first blog post ever, hope it was a good read!
