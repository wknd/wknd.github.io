---
layout: post
title:  "adventures in android flashing prelude"
date:   2018-04-22 12:00:00 +0200
last-modified: 2018-04-22 23:00:00 +0200
categories: blog android nethunter
featured: true
image: 
  name:   "scifi-alley"
  global: true
  
images: 
  aquaris-e45:
    file: "Aquaris-E45-Ubuntu-Edition.png"
    global: local
    description: "Aquaris E45 Ubuntu Edition"
    creator: "ubuntu updates"
    creatorurl: "http://www.ubuntuupdates.com/2015/08/Aquaris-E45-Ubuntu-Edition-Specification.html"
  motorola-moto-g4-plus-r1:
    file: "motorola-moto-g4-plus-r1.jpg"
    global: local
    description: "Motorola Moto G4 Plus official image"
    creator: "gsm arena"
    creatorurl: "https://www.gsmarena.com/motorola_moto_g4_plus-8050.php"

---
I've been using a [motorola G4 plus](https://www.gsmarena.com/motorola_moto_g4_plus-8050.php) for over a year after being disappointed with the direction of [ubuntu phone](https://wiki.ubuntu.com/Touch). I picked the G4 because it was a cheap but capable phone that came with almost completely stock adroid, no bloatware, and the promise of updates.  
But now, its time to upgrade some more to a custom ROM.

## Why did I go back to android
Usually when you see the above title, its about how iOS is inferior and not open enough and a lot of other reasons to pick an android phone over an iPhone. In my case however I've always known that the "it just works" feature of iOS came at the cost of my own freedom to use the device however I wanted, so I never fell into that trap to begin with. 

I've been using android since I got my first [Galaxy S](https://www.gsmarena.com/samsung_i9000_galaxy_s-3115.php), a very expensive phone for me at the time and I kept using it for 6 years. Keeping it usable by switching to cyanogen mod instead of the normal samsung firmware that never got updates. And even using a slightly modified [wifi firmware](https://forum.xda-developers.com/showthread.php?t=2760170) so I got monitor mode working on it (and thus could even crack WEP networks from just my phone).

{% include articleimage.html image=page.images.aquaris-e45 float="right" %}

When it was time to finally upgrade my ancient phone, ubuntu had just released its first official [ubuntu phone](https://en.wikipedia.org/wiki/BQ_Aquaris_E4.5), and I went with that instead of android. I liked the idea of even more opensource components, ability to customize, future device convergence (using your phone as a full fledged ubuntu computer if plugged into a screen and keyboard) and a nicer UI.

And while I think the UI was very nice and intuitive, the other promises were never coming.  
If you wanted to modify it beyond standard phone capabilities you lost your updates, or would lose your changes on update (as an update would simply reimage the entire phone). Convergence was always coming but by the time they got close my phone was no longer a target for it.

But worst of all, and this is what I think caused people to loose interest and eventually was the death of ubuntu phones, was that they absolutely refused to run any background services on the phone. No background services meant the phone's battery would last a lot longer and there was no ability for any apps to spy on you, it was a great privacy improvement.

BUT this also meant no notifications for email, irc, or any protocol that required a persistent connection. When you move to a different app or turned off the screen, every app would disconnect and stop running.  
The only notifications you got was from a few official apps that did have support, or if somehow you got an officially approved background service and it got included into every ubuntu phone. Or lastly their preferred method (and least likely to ever get used) if someone would write their server backend so that it could send push notifications via ubuntu's system instead of whatever native protocol it was supposed to use (and this of course meant every single application would require you to trust a 3rd party with ***all*** your data so it could translate it into something ubuntu could understand, and then trust ubuntu with that data too).  

Needless to say, none of these possible workarounds to the "no background services" rule were ever created. And after a year or more of only using the phone for browsing, telegram and txt messages, I decided that when I upgraded, it would be to go back to android.

## Why moto G4 plus
{% include articleimage.html image=page.images.motorola-moto-g4-plus-r1 float="left" %}

Since deciding to go back to android, I looked around a lot to see which phones offered the best experience.  
Specifically I was looking for something with a dual-sim (the ubuntu phone had dual sim and I bought it while I was in czech republic for a few months which showed the value of that to me), not horrible specs or battery life, but most importantly no bloatware pre-installed.

The moto G4 seemed like the only match for those specifications at the time, they promised to update to android 7.0, and wasn't that expensive either. I decided to wait until they announced the moto G5 for me to buy it, hoping the price would drop even further. But I got a little inpatient and got it right before (though the price didn't really drop immediately afterwards since it was already lower than at its own release). I went with the G4 plus so I could play around with the extra convenience of having a fingerprint reader.

Eventually motorola did release an update to android 7.0 (although much later than when they released in India), and I do get regular security updates (though not as frequent as nexus devices). And right now after some outrage they also prommised to give this phone an update to 8.0.

All in all, I'm pretty happy about this phone. When asked to recommend a phone to people, I always recommend its upgrade (the G5) or a new nokia phone (they seem to be keeping their update and no bloatware promise even better than motorola). The only downside is its lack of compass and so it isn't compatible with things like [google cardboard](https://vr.google.com/cardboard/) (no compass means your gyro will drift and it can't detect magnetic the seal either).

## The Future
After the previous paragraphs you'll understand that I am actually happy with the phone I have now. I have no interest (or budget) to buy a new one (remember, I kept the first android phone running for 6 years!). 

But I will not wait around for the stock android 8.0. I want more control over my device than I have now so I will go with a custom ROM once again, and I want to learn more about the android system while I'm at it.

I will go with [lineageos](https://www.lineageos.org/) for the ROM. In the past I used [cyanogen mod](https://en.wikipedia.org/wiki/CyanogenMod) but they have since gone evil and lineageos is the fork that spawned from that. And importantly, I want to get [nethunter](https://www.kali.org/kali-linux-nethunter/) working on it. 

I've looked up a lot of things and have created a short list of tasks I need to accomplish (in order):
*    Backup important data (only contacts, 2FA data could not be backed up from [Google Authenticator](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) and [FreeOTP](https://freeotp.github.io/) recently lost [that ability](https://github.com/freeotp/freeotp-android/commit/5ced27208fc65999ee5a0103a452877dfb256247), so I'll use [andOTP](https://github.com/andOTP/andOTP) in the future)
*    Unlock the [motorola bootloader](https://motorola-global-portal.custhelp.com/app/standalone/bootloader/unlock-your-device-a)
*    Install custom recovery [(TWRP)](https://dl.twrp.me/athene/)
*    Install [Lineage OS](https://download.lineageos.org/athene) with [google apps](https://wiki.lineageos.org/gapps.html) and [su](https://download.lineageos.org/extras)
*    Create custom kernel based on [the kernel](https://github.com/LineageOS/android_kernel_motorola_msm8952) used by [lineageos-athene](https://github.com/LineageOS/android_device_motorola_athene) with tweaks for [nethunter](https://github.com/offensive-security/kali-nethunter/wiki/Modifying-the-Kernel)
*    Build and test full nethunter package
*    Add pull request to add motorola g4 support upstream
*    Buy usb OTG cable and maybe also usb OTG Y cable

## Conclusion

There are a lot of steps to get where I want to go, and each has a lot of potential problems along the way. Mostly because of my arbitrary requirement of getting [nethunter](https://www.kali.org/kali-linux-nethunter/) running even though my device isn't officially supported(yet!). 

But I'm guaranteed to learn things along the way, and I'm sure I'll manage to keep the phone functional even if I give up at some point.  
Fun times ahead!
