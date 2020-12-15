---
layout: post
title: Setting up gigabit internet at home
category: [engineering]
comments: true
---

![speed](https://github.com/tameravci/tameravci.github.io/blob/master/images/Screenshot%202020-11-23%20182807.png?raw=true)

I made it guys. I really did it. I can now download and upload approximately 125 MB worth of data every second or, put another way, 1000 Mbps. Goodbye to all of the streaming and gaming lag - I am very stoked!

To put things in perspective, this is what I can theoretically do with this kind of internet. I can download:
- a 720p High Definition TV episode — 1GB, in about 8 seconds
- a Blu-Ray Movie — 15GB, in about 2 minutes

So how did I get there?

Well, for starters, I recently moved. I'm still in Seattle, not too far away from the old place. Besides a million other things, moving entails canceling your old internet service and looking for a new ISP at your new place. Even though I didn't really take this into account when leasing this new place, I was pleasantly surprised to hear that they had the Google Fiber service in the new building. It's really not that common - not even in Seattle. It was all setup in the building - it was just a matter of them enabling my unit in the conn room or whatever they have down in the lobby. Anyways, once I activated my service, I realized that my old JCG-150 router from college wasn't ready for this task so I started researching for routers. It's important to note that at this point I had about ~120mbps service because my old router couldn't handle the gigabit internet.

Also a quick note about the cost. It's technically $60 a month but if you pay upfront for a year, it's only about $600 for the first customers. I think, given the reliable speed I'm experiencing, this is well worth it. But.. I'm getting ahead of myself. Initially, I had some other problems.

In the new place, I have an extra room where I setup my office so unlike the old place, the router wasn't going to let my laptop, my TV, and my playstation connect over the wire since they were going to be in two distant spots in the apartment. Slowly, my requirements were shaping up. I needed to work with the network patch panel at the entrance and enable a few of those ethernet ports that were going into my den/office, the living room, and the bedroom. Because the TV and the playstation were going to be right by the same ethernet port, I also needed at least one switch along with a new router.

The google fiber guy told me that I needed a switch if I wanted to enable all the ports in the house. Basically, his idea was to connect the uplink to the switch and then connect the eth cables to the outgoing room ports. Here's the picture that he was thinking about.

![switch](https://lh3.googleusercontent.com/pw/ACtC-3cjafa5J-uhQ7O7wcRJTXoMEs2_R9M7vj87C-HtF5JT6y1K_IywZy7UT2BaNSpg2pvnYnOh2VMjpRIlkFsQrGAa2tI4rONHW1GqlUobM485uuX454bR4jSl0GUy-sccuU-Mid0hjbRz1578C5vaZyYYOg=w565-h937-no?authuser=0)

I definitely did try this setup but ultimately it failed. 
A couple of reasons why this setup wasn't right: 
1. Even though I thought I got a dumb switch, it was actually a smart switch and was getting an IP from the ISP. This prevented my router in the office from getting another IP and so I was left with no connection in the office. This is because most people pay for 1 IP at their homes and it's usually assigned to the router. The router then creates a local network where your devices can join and get a local IP.
2. Even if I'd purchased a dumb (passthrough only?) switch, I wouldn't have used this setup because you still want your PS4 and TV to be behind a firewall. And that's why you should put them in the NAT that your router creates further isolating them from the wild internet. This means, my uplink was going straight into the router.

I ended up configuring my home network as follows:

![home_network](https://lh3.googleusercontent.com/pw/ACtC-3c8t4PL9Udzporz1kcHvjxKGVkF-MS_lZ6TtwkJBMHK-44t-W_d4WJVvASvbOIFvucZJsnntKXHQVxBkrMJbx7u5kAxlYZ03WZ1UIdWM04JE2BicKf_GwTN0KAV-qUAr-cqt6k4EKxBedbWlXTuaCnUsQ=w810-h526-no?authuser=0)

I would summarize the decisions that went into this architecture as follows:
- I ended up buying the TP-Link1500AX router because I wanted to check out the Wifi6 protocol and the 5GHz band. Both paid off because a) wifi devices were significantly faster and b) I rarely have disconnection issues because there's a lot less interference in 5Ghz band. It also provides the bluetooth devices some breathing room because they don't have to do the frequency hopping as much.
- Some of the wifi devices do not support the 5Ghz band so I had to enable the 2.4Ghz band as well so now my router is broadcasting both. Alexa and google devices should upgrade their wifi controller to support this band!
- It's pretty self-explanatory why I'm connecting wired for my main workstation. Wired connections rock. They are reliable and fast. This is pretty much the only station where I am consistently hitting gigabit speeds and it's mind-blowing how fast it is.
- Cat8 ethernet cables were an overkill. I tested with those plus cat5 and cat6 cables - even though cat6+ cables handle the gigabit speed better, I didn't see a drastic difference. Plus who am I kidding - the uplink connection cable from downstairs up to my apartment is a cat5 anyways!
- I still ran into some wifi range problems! Sad, but it happens. Even though the wifi6 protocol is supposed to be better at handling this, because I moved to a larger apartment, we still have no signal in the bedroom from the main router that is located in the network panel by the entryway. Luckily, I didn't get rid of the old router so I turned that into an access point and literally placed it on the other side of the apartment. For some reason, my pixel 2 still gets the signal in the bedroom but my girlfriend's iphone prefers the AP and so does the Alexa. Both users (devices?) don't care much about speed. It's fast enough for them so I managed to satisfy their strong signal requirement.
- I used the tp link sg80e gigabit switch in the living room to connect the Ps4 and the TV. I confirmed that the ps4 is pulling gigabit speed but not the samsung TV. Nothing I can do about that. PS4 matters more because we do a lot of upload over there when playing multiplayer games. Nowadays, both us-east and us-west game servers show very small latency for me.
- Another interesting thing that's set up in my network is the Pi-hole DNS server. This is an open-source tool that you can install on your rasperry pi or some other linux based computer. It's super convenient as you can set this up at the network level so the internet is already filtered ad-free before going to your end clients (devices). I kind of whitelisted my main workstation from this DNS proxy because it was leading to a considerable decrease in speed. I highly recommend using this tool if you are bothered by the ads and you realized you can't really install an adblocker on your "Google" phone or your "Samsung" tv. If you don't want to create a default proxy, you can also configure each device individually to pick and choose. You'll be surprised by how many ads we are bombarded with.. Don't get your hopes high though. Some companies are serving the ads from the content endpoints so essentially that's defeating the pi-hole's capability to block ads. For example, you can't watch Hulu at all.. You have to whitelist their endpoint and that opens the gates for the ads. Youtube is like 50/50. There's definitely a decrease in the number of ads I see but they must be doing something clever that doesn't block the ads on the TV entirely. Strange that the ABP on the browser extension works much better in this regard.
- Finally, I would like to praise the TP-link router software. Super easy to use and intuitive. The firmware upgrades were also fast and reliable. And best of all, they provide a free Dynamic DNS service. With this, you can take advantage of the free service they provide and slap a domain on your dynamic IP! Open your gaming ports, and voila, you can host the game servers to play games with your friends all over the world or host your own website for free but do it at your own risk :-) The only limitation is that they can only bind one domain at a time. Which is fine with me.

Hope this was helpful to someone out there. Let me know if you have any questions about the setup.

Follow my twitter for more updates:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Signed up for a gigabit <a href="https://twitter.com/googlefiber?ref_src=twsrc%5Etfw">@googlefiber</a> at my new place and even got a few cat8 cables for the switch. This is already a huge jump from my old setup but I still wonder why the download speed is ~30% less than the upload 🙃 <a href="https://t.co/u85bTX2OrC">pic.twitter.com/u85bTX2OrC</a></p>&mdash; Tamer Avcı (@tamer_avci) <a href="https://twitter.com/tamer_avci/status/1305319415634821120?ref_src=twsrc%5Etfw">September 14, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
