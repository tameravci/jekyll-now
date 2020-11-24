---
layout: post
title: Setting up gigabit internet at home
category: [engineering]
comments: true
---

I made it guys. I really did it. I can now download and upload approximately 125 MB worth of data every second or put another way 1000 Megabits per second. Goodbye to all of the streaming and gaming lag.

To put things in perspective, this is what I can theoretically do with this kind of internet. I can download
- a 720p High Definition TV episode — 1GB, in about 8 seconds
- a Blu-Ray Movie — 15GB, in about 2 minutes

So how did I get there?

Well, for starters I recently moved apartments. This obviously entails canceling your old internet service and looking into new ISPs supported at your new place. Even though I take this into account when making the decision to sign a lease at this new place, I was pleasantly surprised to hear they had Google Fiber service available. It was all setup in the building, it was a matter of them enabling my unit in the conn room or whatever they have down in the lobby. Probably a giant modem that splits the incoming bandwidth into the units. Anyways, once I activated my service, I realized my old JCG-150 router from college wasn't ready for this task so I started researching. It's important to note that at this point I had about 120mbps service because my router couldn't handle the gigabit internet.

Another problem that I realized was that I had an extra room where I setup my office so unlike the old place, the router wasn't going to be able to let my laptop, my TV, and my playstation connected over the wire since they were going to be in two distant spots in the apartment. Slowly, my requirements were shaping up. I needed to work with the network patch panel at the entrance and enable a few of those ethernet ports that were going into my den/office and the living room. Because the TV and the playstation was going to be right by the same ethernet port, I also needed at least one switch along with my router. 

The google fiber guy had told me I needed another switch if I wanted to enable all the ports in the house. Basically, his idea was to connect the uplink to the switch and then connect the eth cables to the outgoing room ports. Here's the picture that he was thinking about 

![switch](https://lh3.googleusercontent.com/pw/ACtC-3cjafa5J-uhQ7O7wcRJTXoMEs2_R9M7vj87C-HtF5JT6y1K_IywZy7UT2BaNSpg2pvnYnOh2VMjpRIlkFsQrGAa2tI4rONHW1GqlUobM485uuX454bR4jSl0GUy-sccuU-Mid0hjbRz1578C5vaZyYYOg=w565-h937-no?authuser=0)

I definitely did try this setup but ultimately it failed. 
A couple of reasons that this setup wasn't right: 
1) Even though I thought I got a dumb switch, it was actually smart switch and was getting an IP from the ISP. This prevented from my router connecting to the port in the office from getting an IP hence I was left with no connection in the office.
2) Even if I purchased a dumb (passthrough?) switch, I wasn't going to stick to this because you still want your PS4 and TV to be behind a firewall. And that's why you should put them in the NAT that your router creates further isolating them from the wild internet.

I ended up configuring my home network as follows:

![home_network](https://lh3.googleusercontent.com/pw/ACtC-3c8t4PL9Udzporz1kcHvjxKGVkF-MS_lZ6TtwkJBMHK-44t-W_d4WJVvASvbOIFvucZJsnntKXHQVxBkrMJbx7u5kAxlYZ03WZ1UIdWM04JE2BicKf_GwTN0KAV-qUAr-cqt6k4EKxBedbWlXTuaCnUsQ=w810-h526-no?authuser=0)

I would summarize the decisions that went into this architecture as follows:
- I ended up buying the TP-Link1500AX because I wanted to check out the Wifi6 protocol and the 5GHz band. Both paid off because a) wifi devices were significantly faster b) I rarely have disconnections and there's a lot less interference in 5Ghz band and it also provides the bluetooth devices some breathing room.
- Some of the wifi devices do not support the 5Ghz band so I had to enable the 2.4Ghz band as well so now my router is broadcasting both. Alexa and google devices should upgrade their wifi controller to support this band!
- It's pretty self-explanatory why I'm connecting wired for my main workstation. Wired connections rock. They are reliable and fast. This is pretty much the only station where I am consistently hitting gigabit speeds and it's mind-blowing.
- Cat8 ethernet cables were an overkill. I tested with those plus cat5 and cat6 cables - even though cat6+ cables handle the gigabit speed better, I didn't see a drastic difference. Plus who am I kidding - the uplink connection cable from downstairs up to my apartment is a cat5!
