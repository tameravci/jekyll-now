---
layout: post
title: Building BAIL app's backend
category: [engineering]
comments: true
---

I saw a tweet the other day and it really piqued my interest. Here's the source to give her credit: https://twitter.com/mattiekahn/status/1224100727578120193

"The app is called “You’re Cancelled.” When you’ve made plans that you wish you could cancel, you go into the app and press a little button. If the other person presses theirs too, congratulations! Confetti exploded and your plans are cancelled." 

Needless to say, I was immediately sold. I was drinking beer when I saw it so I told my friend to hold my beer. Honestly, the idea might promote something anti-social but I just thought it would be a good exercise to flex my android app as well as in general side project skills. I fully intend to finish this and release it on iOS and Google Play store if I can find a solid designer who can finish the UI for me. I will get the rest done. So if you like to build cool things and have experience on Android/iOS design, please inquire within.

Without further due, let's get right into it. First things first. Pick a playlist. Honestly, if I'm coding something exciting but also kinda edgy, I'll go with the Social network soundtrack. Enjoy: https://open.spotify.com/album/1ijkFiMeHopKkHyvQCWxUa?si=3SGegSNkTom9JHvelWcyQg

Next is obviously alcohol. But I won't take "hold my beer" back. Instead, let's get funky. Pour some kahlua with club soda. And lots of ice. 

Alright, ladies and gentleman we are ready to code. A junior engineer would say. First, we need to design the interface, use cases and the data model. The rest, we can just hand-off to a junior engineer.

Let's start with our requirements aka the use cases. There might be a slight difference between these terms but it's ok.

```
1. Authenticate the user of the application
2. Secretly enter the plans that you wish you could cancel
3. View all the (unconfirmed) plans that you entered
4. Be nofitied when the cancellation desire is mutual! Celebrate the mutual cancellation of plans on your own.
```

That's it. We got to keep it simple because, you know, I have a day job. Also starting simple and getting feedback is perfectly fine. Reduce the scope. Feature down. Underdo your competition. *Winks at 37signals, you guys know it.

Alright so now that we know our requirements, let's make them crystal clear from the technical standpoint. This is where the engineering team usually chimes in. We take the "more abstract" requirements and turn them into tangible technical capabilities.

Security is a huge thing and be able to authenticate users is essential for almost all applications. We can't take many shortcuts here. However, I will go with the simplest approach here. Normally, I hate SIM cards and phone numbers but they do simplify things. I hate social-media app based auth models even more so I'll go with the phone numbers.

Obviously the right thing would be to interact with the UICC SIM card, maybe retrieve IMEI etc. but ain't nobody got time for that. We will do the second most feasible approach. One-time OPT codes sent via SMS. Here's the API we are going to build.

```
Given a phone number, generate a one-time use time-based OTP, send it via SMS and ask the user to enter the OTP on the screen
```

or in API form
``` java
/*
Generates an OTP, stores it with a TTL for a phone number auth attempt
*/
void requestOtp(PhoneNumber number);
```

Next, we need to be able to retrieve the auth token OTP and complete authentication. After this point, the user should be able to use the app until they switch to a different device. Obviously, we can limit the client auth to a certain period (3 months?) and re-ask them to do the challenge.

``` java
/*
Given an OPT and a phone number, authenticates the user or throw AccessDenied (if auth fails or time has expired upon which the client should retry the flow)
*/
ClientId authenticate(PhoneNumber number, OTP authToken) throws AccessDeniedException;
```

Alright, assuming we can authenticate users, we can now focus on the core use case. Here it is extremely critical to agree on the time. I like to go with the granularity level of daily plans. I mean, who makes a plan with the same person more than once on any given day. So our app will allow you to secretly enter your plans on a daily basis and with however many people.

This means, our date consists of 2 questions: 1) With whom (contacts prompt) 2) When? (calendar prompt). Simple.

```java
/*
Given a date and a contact, stores the plans that the user wishes they could cancel
*/
void schedule(ClientId authenticatedAndAuthorizedUser, Date when, PhoneNumber withWhom);
```

It's important to note that all these APIs will have to carry the authenticated user id so that the backend can check whether or not this user is authorized to set a date on behalf of the owner of the client id. Usually, those client ids or auth tokens should be short-lived and periodically refreshed. Ignoring for the time being.

We are almost there. We need one more API. Whenever I open the app, I'd like to see my unconfirmed plans. Remember, we are sending notifications when both parties enter their plans that they wish to cancel but otherwise the plans are still happening and you will hopelessly look at your dashboards. For that, we need to retrieve the unconfirmed plans from the backend starting from today into the future

```java
/*
Given a user retrieves all the future plans in paginated form
*/
List<Plan> getPlans(ClientId authenticatedAndAuthorizedUser, NextToken nextContinuationToken);
```

where ```Plan``` is
```java
public class Plan {
User withWhom;
Date date;
Status status; // (enum: UNCONFIRMED, CANCELLED etc.)
}
```


This API will query all the future unconfirmed and confirmed(?) plans that the user has with others. The continuation token is set if the result size is bigger than X. Let's not worry about pagination just yet but yeah it's obviously gonna be there in this API.

We can build another version of this for archived cancelled and not cancelled plans that the user has for, you know, bragging points. What else do we need?

We need to be able to send notifications only when both parties cancel on each other. I love the confetti idea. So what I'm thinking is when the feelings are mutual, we should send an SMS notification with a link to open the app, and then boom, the app shows you what plans are cancelled, when & with whom etc.

```java
/*
Given a notification token retrieved via SMS and the authorized user, retrieves the cancelled plan details to view.
*/
Plan getCancelledPlan(ClientId authenticatedAndAuthorizedUser, NotificationToken token);
```

We all love push notifications but that's more work. Here we will take advantage of the SMS integration and will force the app to make a backend call to retrieve the just cancelled event. Similarly, we need another API to retrieve all the notifications so that if they miss the SMS or something, they can go back to the app and hit on the notification page and go from there.

One thing that's critical is to NOT send the notification immediately AFTER the second person puts in the plan details. That would make it obvious to both parties who entered the details first. Which is why we should be careful around building that core logic in that we introduce some jizzer between the time plans are mutually confirmed and the time notificatins are sent out. The exact jizzer should depend on the time that's left until the date. Nothing difficult, meh, implementation detail even but the idea is essential to success.


