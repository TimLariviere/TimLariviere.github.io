---
layout: post
title: "One Mac. Small Xamarin team. How to. Part 1"
date: "2016-10-21"
---

When targeting iOS, whichever technology you choose (ObjC/Swift, Cordova, Xamarin, etc.), Apple requires you to build your application on a physical Mac machine. Running MacOS in a virtual machine on a non-Apple machine is not an option, as it is forbidden by Apple.

In this serie, we will look at the available options for a small team of Xamarin developers (up to 5 devs). This post is a brief look at the possibilities offered to us in terms of hardware when developing for Xamarin. Then in the next two posts, we will take a deep dive at the last option, one Mac for a really small team, and how to configure it.

**I don't have any Mac, so what are my options when targeting iOS with Xamarin ?**

Well, there are a few ones. But let's be honest, you **have to** buy at least one Mac.

**1) Get each developer a Mac as a primary workstation.**

This is the most widely used option when working with Xamarin, even the creators of Xamarin have used and are still using MacBooks Pro.

Having a personal Mac has multiple advantages : you have access to the most advanced and stable tools for Xamarin with Xamarin Studio on MacOS. The machine is able to build and run iOS apps directly on a simulator or on a real device. It's the equivalent of developing Windows apps with Visual Studio.

If needed, you can use XCode; Xamarin Studio completely supports XCode to create your storyboards. And last but not least, contrary to MacOS virtual machine restriction, you can have a smooth running Windows on your Mac, and have access to its tools (Visual Studio, etc.).

It is perhaps the most expensive, but it is definitively the fastest and comfiest option for your team.

**2) Get each developer a Mac as a secondary workstation**

If you don't want to buy a pricey MBP to each developer, you can settle for a Mac mini.

Besides being less powerful and not really fit for heavy usage by developers, you have access to one useful capability of Xamarin plugin for Visual Studio : Remote debugging. That way, developers can still use their main workstations with Visual Studio and, while Visual Studio itself can't build and debug iOS apps, the Xamarin plugin will connect to a Mac (with Xamarin installed) through the network to remotely build and debug right from Visual Studio !

![MacAgent](/assets/2016-10-21-one-mac-small-xamarin-team-how-to-part-1/MacAgent-300x278.png)

So you only need a powerful enough Mac to sustain building and running apps for Visual Studio. The Mac just need to be up and running with one user session open.

This option has the advantage of not depending on the developer. If your company assigns one machine per developer and one of your team member leaves but still remains in your company, having a secondary Mac mini is easier to reassign than a main workstation.

And if you truly need to do something specific with the Mac (like using XCode), you can.

So this is a cheaper option, but the remote debugging really slows things down. Your team won't be as efficient as with the first option.

**3) One Mac to rule them all**

The Xamarin daemon on MacOS, allowing for the Xamarin plugin for Visual Studio to connect, is quite impressive. It allows for multiple developers to connect to a single Mac with a single user session and can handle multiple builds simultaneously.

The single user session has a few shortcomings. All developers share the same user session and if two or more of them need to use the iOS Simulator or XCode, they can't do it at the same time. If the developers need to access XCode, their works are not isolated.

**But** there's one neat feature integrated into MacOS to overcome those shortcomings : remote access. The Mac has an integrated VNC server allowing for multiple user sessions running simultaneously. This way, you can isolate access of the Mac for all devs and Xamarin works really well in that mode : Remote debugging is still usable (and asking the developer's personal session) and the iOS Simulator is available to all devs at the same time !

You can even add your Mac to an Active Directory to manage your users.

With this option, you can take only one top end Mac mini or a Mac Pro for your whole team.

**Wow, this looks promising. What do I need to do ?**

You can head to the [second part of this series](/2017/03/14/one-mac-small-xamarin-team-how-to-part-2.html). It will lead you on every steps needed to set up the Mac (connect it to an Active Directory, allow remote access and multiple sessions) and your developers' machines.
