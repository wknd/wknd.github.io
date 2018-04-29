---
layout: post
title:  "shared USB switch"
date:   2018-04-29 012:00:00 +0100
last-modified: 2018-04-29 13:00:00 +0100
categories: blog electronics pcb
featured: true
head-image: 
  name:   "keyboard-usb"
  global: true

images: 
  custom_diagram_3_SM74611:
    file: "custom_diagram_3_SM74611.png"
    global: local
    description: "Diagram of a smart diode"
    creator: "Texas Instruments Incorporated"
    creatorurl: "http://www.ti.com/general/docs/datasheetdiagram.tsp?genericPartNumber=SM74611&diagramId=SNVS903B"
  hub-reference-design:
    file: "hub-reference-design.png"
    global: local
    description: "Schematic of evaluation board for USB2512"
    creator: "SMSC Austin"
    creatorurl: "ftp://ftp.smsc.com/pub/usb/evb2512q36bas.pdf"
  toggle-switch:
    file: "toggle-switch.jpg"
    global: local
    description: "A nice big toggle switch"
    creator: "Jinghan switch electronic CO.,LTD"
    creatorurl: "http://www.jhswitch.com/html_products/On-On-Toggle-Switch-E-SG1321-203.html"
  

---
I have a problem, a first world problem. It is actually two problems that are solvable together. For one, my desk is too small. And secondly, the keyboard on my little tablet is pretty bad. I think to solve this I want to hook my keyboard up to a USB sharing switch so I can use my main laptop and the little tablet at the same time with a single keyboard.  
Of course I could just buy it, but then I thought it would be more interesting to make one myself.

I'll try my best to describe my setup; I have a small corner desk, with my main laptop on the left and my bigger screen sitting on top of an old lab powersupply in front of me. This leaves me just enough room for my [tenkeyless keyboard](http://www.duckychannel.com.tw/page-en/Ducky-One-TKL/) and a [mouse](https://www.logitech.com/en-us/product/mx-anywhere-2s-flow). 

I picked the keyboard specifically because it was the only [tenkeyless](https://deskthority.net/wiki/Tenkeyless_keyboard) keyboard that came in my key layout (azerty-be). And I picked the mouse because of its scrollwheel and because it had the ability to be used with up to 3 different computers at the same time. Although sharing the mouse is mostly useful when actually switching machines, not so much for using them at the same time. But that scrollwheel is great for going through large chunks of code and makes it kinda fun to go through it.

But sometimes I'll have both laptop screens occupied, or I'll even play a game. And when that happens I like to place the little laptop in front of my keyboard and under the big screen to keep my various instant messaging applications.  
The problem occurs when I want to reply to a message. I have to reach over my keyboard, to the other even smaller keyboard and type. Or even use its touchpad(which is partially under my keyboard) to naviage to the proper window. That just won't do.

## Solution
The solution as mentioned is a USB sharing switch. A little device with 2(or more) USB inputs, and 1 output. Often controlled with a button on the device, it lets you electronically unplug your USB device from one computer and plug it into another.

They come in many different variations. You could have a button to control it, it could read your key presses and switch by a key combination, it could have more than 2 inputs, it could have an integrated usb hub, etc etc. It is basically a KVM but without the video (my screen only has 1 hdmi input, so it'd be nice to have something to switch video too. But that makes it way more expensive).

How they operate internally can also differ. 
And as mentioned, I won't be buying such a device, I want to make it myself.

## Design

Luckily we don't have to design every component ourselves anymore. There are enough USB applications that require this sort of switching that manufacturers have created IC's specifically to handle switching the high frequencies signals used by USB 2.0.  
Though I should note that some newer RGB keyboards use USB 3.0 to be able to handle a larger current draw, which would probably not be compatible with this method. My keyboard however is still just USB 2.0 and I don't plan to buy another one any time soon.

##### Switching data lines
There are several way variations in how USB sharing switches work. Some (most?) have an analog mux to act as if you were physically disconnecting the device from one computer and plugging it into another.  
In others your keyboard and mouse might actually be connected to an internal microcontroller, while that controller is connected to both your computers. This has the advantage that you could switch faster(since your device is never "unplugged"), and possibly control it with your keyboard. The downside is that it'll only be able to handle standard HID devices. But there would also be an increase in input lag since first the microcontroller has to poll for the keypresses before the computer can poll and detect them itself.

High input lag is not acceptable at all and I'm not sure my keyboard counts as a standard HID device. My keyboard has [n-key rollover](https://en.wikipedia.org/wiki/Rollover_%28key%29#n-key_rollover), depending on how it's implemented this could mean my keyboard presents itself as multiple devices. So while those are all standard HID devices, the microcontroller would have to be able to listen for all of them, and present itself as multiple devices too.  
So the analog mux solution seems best for my situation.

I will limit my design to switching between 2 devices, and I think [this IC](http://www.onsemi.com/PowerSolutions/product.do?id=FSUSB42) will work for it.

A future iteration might use a microcontroller to detect keypresses and switch via keyboard commands. But instead of having that controller sit between the datalines, I'd rather have it passively spy on them instead. That is outside of the scope of this project though, first I just want a basic device that works.

##### Switching power
Each device will have a slightly different USB voltage. Even if they were designed to be exactly the same, differences in cables type, length, even how well it was plugged in will cause variations. So we can't just tie the power rails together or one device will be feeding power into the other in a manner it wasn't designed for.

There are several ways we can solve this problem. We could use an external powersupply, which also means we can put high loads on our devices without affecting the computers. We could also manually switch the powerline from one or another when we switch. 
Or we could "[OR](https://www.eetimes.com/document.asp?doc_id=1273175&page_number=2)" the powerlines together with some diodes. That way it'll draw power from whichever computer can deliver the highest voltage.

{% include articleimage.html image=page.images.custom_diagram_3_SM74611 float="right" %}

I do not want to have to use an external power supply, so I don't think that is an option (though I'm okay with optionally using an external power supply).  
Switching the powerline manually is an interesting idea, especially with an included delay so the device gets powered off. But this might need more advanced signal switching too so the signal gets "plugged in" after the power is turned on.  
OR-ing the powerlines together is the simplest (and maybe most robust?) solution, but if we use regular diodes the voltage drop across those would be too high. Schottky diodes would have a lower voltage drop and may work. An alternative are so called smart diodes. They are a combination of a diode, a fet across it and some detection and control system. So when the fet is powered the voltage drop is much lower than with just the diode.

Since I don't want to include a microcontroller yet, I think OR-ing the powerlines together is the best choice right now. Though having it optionally switch off power from the computers if there is an external power applies is an interesting choice and shouldn't require an extra microcontroller. Since I hope to also include a (very)small USB hub in this, it would let my device function as a powered hub.

There are a bunch of [devices](http://www.ti.com/power-management/oring-and-smart-diodes/overview.html) build for power OR-ing and/or switching so we don't have to create our own. Some have a built in FET and others are controllers that require an external FET. 

If I go with the [tps2113a](http://www.ti.com/product/tps2113a/description) then I can configure it to prefer input power from the first input over the second. This way I can make sure it draws power from the laptop if it is plugged in, or if using the device as a usb hub it can draw power from an external supply on port 1. Depending on which MUX I choose for the data lines, I can even have it detect when a charger is plugged in on port 1 (as opposed to a device that has data) and automatically switch the datalines to the other port.

##### USB hub
Even though my mouse has its own built in functionality to switch to different computers, it would be way more convenient if the keyboard and mouse switch at the same time. Because of this I want to incorporate a small usb hub into the device so one input can control at least 2 devices. A device such as the [USB2512B](http://www.microchip.com/wwwproducts/en/USB2512B) would be suited, and has additional features to set specific ports as a device charging port.  
Perhaps if I'm detecting a charger on input 1, I can enable charging on output 1 too. Though I think that'll require an additional USB power controller to do properly and I'm not sure yet if I want to go that route.  
An interesting idea is also to reset the hub when switching inputs, this way we wouldn't have to wait for the hub to figure out it got disconnected and instead only have to wait for it to start up again (500µs for the example chip).

I found a nice [reference design](ftp://ftp.smsc.com/pub/usb/evb2512q36bas.pdf) of this chips previous iteration, and will refer to this again when I make my design. 
{% include articleimage.html image=page.images.hub-reference-design %}

##### Controlling input
{% include articleimage.html image=page.images.toggle-switch float="right" %}
To control the input I want to use a push button to trigger the change. This should cause a control line to toggle which controls the MUX.  
However, I also want to be able to control it with a large external toggle switch.

The reason for the toggle switch is simply because it feels nice to toggle them and it can be mounted in a convenient place (perhaps next to my screen). Just look at that switch in the picture, doesn't that look like I'd be extremely satisfying to toggle?  

I might need some extra circuitry to support both a button on the device and a toggle switch, and an extra connector to plug it in. We'll go over that in more detail in another post.

##### Port types
Since no one has old bulky USB A to USB B cables lying around anymore, I think its best if the inputs use microUSB connectors. These cables are found almost everywhere, and we can use this opportunity to connect the pins in such a way that it also acts as an [USB OTG](https://en.wikipedia.org/wiki/USB_On-The-Go) connector even if a non OTG cable is used (are there even microUSB to microUSB cables that aren't OTG cables?).
The output ports should be normal big USB 2.0 type A connectors.

##### Specific IC's
I've mentioned example chips from several manufacturers, but I'll have to evaluate which specific ones I'll go with as I'm designing it. There might be some benefit to picking everything from a single source, or maybe I'll just pick whatever is cheapest or has the coolest functionality.

When I select these chips I will try to take future revisions into account so I can use the same parts in different versions.

##### Housing
The completed circuit board should have some sort of housing to keep it safe. I have no idea what I'd use for that at this point, hopefully I'll figure out something nice.

It is safe to say though, that it'd be a lot easier for me to design the [PCB](https://en.wikipedia.org/wiki/Printed_circuit_board) around a specific enclosure than it would be to modify or build my own enclosure. 

## Modes of operation
Now that I know what will go into a design I can start to describe the desired modes of operation. Since its a design by me, for me, it could always change while I'm creating it. But this is as close as we'll ever get to a design specification.

##### USB hub
Only one input is plugged in.

It grabs its power from the input plug, and works as a USB hub. Random things can be plugged in there and toggleing the switch should do nothing.

##### USB switcher 
The original design goal. In this mode, both input ports are plugged into USB hosts.

It gets its power from the first input port and delivers that via the built in hub to the devices.  
By pressing the onboard button or toggling the external switch we can switch which input port is used for data.
  
When we switch we also reset the USB hub and this should hopefully reconnect them faster. (I want to make resetting the hub optional in my design so I can evaluate the difference)

##### Power splitter
A charger is plugged in on input port 1 or 2 and nothing on the other.

The power is transfered via the hub to whatever is plugged in there.

##### Powered USB hub
A charger is plugged in on port 1 and an other device on port 2.

The power is transfered via input port 1 to whatever is plugged into the hub and data is transfered via port 2 to the hub.  
Pressing the button or toggling the switch does nothing.

##### Complication
Working as a powered or non powered USB hub and disabling the toggling could only work if I can detect that the charger is on port 1. If I can't detect a charger on a specific port then we can't disable toggling and a user(me?) might accidentally disconnect his devices.  
Though this isn't really a big deal and would simplify the design.

## Conclusion
It is definitely possible to build this myself, and though it may not be cheaper (especially when counting tax and shipping costs on parts) It will be a lot more interesting than buying something off the shelf.

If I reach all the goals set out here I'll have a nice solution to my small desk problem. And accidentally I'll have a USB OTG hub, with optional external power too! Very useful for the android stuff of previous posts.

Stay tuned for more updates on this subject.
