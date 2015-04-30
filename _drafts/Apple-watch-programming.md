---
layout: post
title:  "Programming for Apple Watch"
date:   2015-04-30 15:38:56
author: Mathias Claassen
categories: Apple Watch
author_id: mathias

---



Apple Watch is here.
Apple say it is their the most personal device yet to appear. It adds a new dimension to the tech world and it is just a glance away. Yet the most excitng feature it offers might be the Digital Touch which is designed to bring you even nearer to your friends.

But seeing it from a Programmers point of view, how does Apple Watch differ relating to other devices. Which apps should have a Watch extension and which are the most important concepts that should be clear when deciding to create an Apple Watch app? 

At a glance
-----------

To start with, you will surely know that the Apple Watch needs to be near your iPhone to work. This is because the processing of the app is done on the iPhone. So every app has two parts, one on the iPhone and one installed in the Watch.

The framework used to create Watch apps is WatchKit. Therefore the iPhone part of an app is called WatchKit extension. Each app will have a WatchKit extension ruuning on the background of the iPhone processing the information it gets from the Watch. The part on the Watch itself is called WatchKit App which just consists of some user interface resources like storyboards.

What are the Watch's most interesting features?

* ###Connection
The Apple Watch is really connected to the user mainly because of the Digital Touch. This will create a new way of communicating and the Watch will certainly soon be associated with it. This creates an atmosphere that third-party apps should respect and if possible support according to the Apple team. 
But up to now WatchKit apps cannot access the sensors of the Watch which will limit how personal they can get with the user.
* ###Glances
A glance is a read-only view and it should show the most important information of an app. The user cannot interact with them and if he taps on a glance then the app will be launched, where the user can interact. Glances are inserted in the apps bundle and go in the same storyboard but they have a particular controller whose only responsibility is to set the information on the screen.
* ###Notifications
Notifications are a key feature in Apple Watch as the watch is handier to look at than an iPhone. The system provides basic notification support so that the apps do not have to support it but they can create their own notifications customizing the content and style. The user can launch the app or take action directly using the action buttons. These buttons save the user some time as he can avoid launching the app just to tap a button.
There are two types of notifications: short-look and long-look. At first, the short-look interface will be shown but it will then transition to the long-look interface when the user continues looking at it.
The short-look shows the user the basic info like the apps name and the title of the notification and this cannot be customized. The long-look on the other hand does also show the whole notification string and the associated action buttons and can be customized.


The Watch is also great for using Siri and fitness tracking apps among others.

Which apps should have a Watch extension?
----------------------------------------

Apps that support notifications well might be interested in letting their notifications appear on the watch. As I mentioned above it is very handy for the user to see his notifications on his wrist without need to get the iPhone from the pocket. But there is also the counterpart that for its small size it can fastly be overfilled with notifications. If there are many apps installed that send notifications then the users will certainly get annoyed for constantly looking at the notifications. So the apps should really consider which notifications are right to be sent to the watch.

Apps that have a very simple interaction where the user needs very few taps to get what he wants are also suited to the Apple Watch. But, on the other hand, apps that rely on a series of actions from the user, possibly with text input or similar will should not bother making a Watch App as a smartphone is perfect for their needs. All this comes because of the fact that the small size of the Watch does not allow complex interactions which would be a total mess if not completely impossible.


What do I have to know before starting to program WatchKit Apps?
-----------------------------------------------------

If you are an experienced iOS programmer you will be familiar with most of WatchKit's stuff but there are several differences, some things that cannot be done and others that are much easier to do. Also there are these new concepts like `glances` and that utterly small screen size.

####Differences when coding

To begin with, the WatchKit extension and the main iPhone app are different processes on the iPhone so that if they have to communicate then there has to be a special means for it. And obviously there is one but we will see that later. First we have to think about the fact that the WatchKit app will be a summarized version of the iPhone app so that we will almost certainly want to reuse some code. The approach to be taken in this case is to create a framework to be included in both apps (iPhone and Watch) with all the common code. This framework should contain all the classes and the general helper methods that are used on both apps.

Managing tables, rows and segues in WatchKit is generally very similar to iOS but easier to use. For example, you do not need worrying about data sources and delegates for tables in WatchKit. Furthermore, there is no cellForRowAtIndexPath so you just set all the rows in the `awakeWithContext` method once and for all. This is the method where the whole interface should be initialized. 

The segues have the particularity that passing data to the destination controller is easier allowing more concise functions. The function to override is:

{% highlight objc %}
func contextForSegueWithIdentifier(segueIdentifier: String, inTable table: WKInterfaceTable, rowIndex: Int) -> AnyObject? 
{% endhighlight %}

where you have to return an object that will be used as context for the destination interface controller.

####Storyboards

It is also very recommended to use storyboards in WatchKit. This is because fonts and positioning of elements cannot be set from code and setting their color is also handier from the storyboard. The positioning of elements, in fact, is completely different to iOS. In WatchKit everything has to be pinned to something: top, bottom, right, left of the screen or to another element.

There is no Autolayout in WatchKit. This means that instead of defining constraints you will now define margins and the relative ordering between elements. In addition, all elements are ordered from top to bottom so that there cannot be two elements in the same horizontal line unless they are in a group. Groups are the element containers that allow us to define any kind of layout as groups can attach elements horizontally or vertically and can obviously contain other groups.

Linking the storyboard to code works just the same as in iOS even though the storyboard resides in the WatchKit App on the Watch and the controllers are on the WatchKit extension on the iPhone. Apple handles this so you can use outlets and actions just like always.

####Connecting everything

There is, of course, a lot of interaction between the different parts of such an app. First the glances have to communicate with the WatchKit App and then the WatchKit App has to interact with the WatchKit extension on the iPhone. And last but not least, the WatchKit extension might want to request some data from the main iPhone app. Connecting the WatchKit App with its extension on the iPhone is done by Apple so that developers have not much to do here but consider that it is being done using Bluetooth.

Let's start with sending information from glances to the WathKit app.
Glance Interface controllers are just responsible to display the correct information which should be done in the `awakeWithContext` method and that is also the place used to call the method `updateUserActivity` that will produce the connection to the app. This method informs WatchKit that there is a User activity going on and it allows you to pass some data to your app in a dictionary.
This User activity event is handled by the app by overriding:
{% highlight objc %}
func handleUserActivity(userInfo: [NSObject : AnyObject]!) 
{% endhighlight %}

There are two functions used to connect the WatchKit App with the main iOS app, via the extension on the iPhone. The first function is used by the WatchKit extension to call the main app on the iPhone to make a request to which that app will reply with the information the WatchKit extension needs. The other function is used to define this reply. It is important to note that calling the main app will not launch it on the iPhone but just make it run in the background.

So to request data from your main app on iOS, in your WatchKit extension controller(s) you will invoke:

{% highlight objc %}
(BOOL)openParentApplication:(NSDictionary *)userInfo reply:(void (^)(NSDictionary *replyInfo, NSError *error))reply
{% endhighlight %}

And in the AppDelegate of your main iOS app you must then attend this request:

{% highlight objc %}
(void)application:(UIApplication *)application handleWatchKitExtensionRequest:(NSDictionary *)userInfo 
									reply:(void (^)(NSDictionary *replyInfo))reply
{% endhighlight %}

####General considerations
The Watch will communicate with the iPhone over Bluetooth and it will do it for every user interaction. This has to be taken into account as it may be error prone and slow. Caching data and background fetches can be very helpful here.

When you are designing a WatchKit App you should not try to replicate your iPhone apps design. The Watch is so much smaller and it has a different functionality so re-design your app to fit the new device.

Finally, WathcKit is pretty new and so it still has a lot of bugs and constraints for developers. One of them is that it seems a WatchKit App target must be iOS 8.2 (not 8.3 or whatever) for now.


<!--
* * Use storyboards. Fonts and positioning can't be set from code. Coloring easier from storyboard.
* * Positioning: Everything is pinned to something. Order and position. No autolayout.
* * Groups to set all kinds of layouts.



* * Frameworks to reuse code in iOS and Watch
* * WatchKit is generally easier when working with tables, rows, segues, etc.
* * Set up interface in awakeWithContext.
* * TableView: set all rows at once, no datasource, no delegate.

* * Apple handles remote actions and outlets.
* * Connection between glance and WatchKit App.
* * Connection between extension and main App.

* * WatchKit App target must be iOS 8.2 for now ...


References:
https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/WatchHumanInterfaceGuidelines/index.html
https://developer.apple.com/library/ios/documentation/General/Conceptual/WatchKitProgrammingGuide/
http://realm.io/news/watchkit-mistakes/
-->


