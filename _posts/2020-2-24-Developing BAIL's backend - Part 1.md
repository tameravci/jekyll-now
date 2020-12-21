---
layout: post
title: Building BAIL app's backend - Part 1
category: [engineering]
comments: true
---

I saw a tweet the other day and it really piqued my interest. "The app is called “You’re Cancelled.” When you’ve made plans that you wish you could cancel, you go into the app and press a little button. If the other person presses theirs too, congratulations! Confetti exploded and your plans are cancelled." 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">The app is called “You’re Cancelled.” When you’ve made plans that you wish you could cancel, you go into the app and press a little button. If the other person presses theirs too, congratulations! Confetti exploded and your plans are cancelled.</p>&mdash; mattie kahn (@mattiekahn) <a href="https://twitter.com/mattiekahn/status/1224100727578120193?ref_src=twsrc%5Etfw">February 2, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Needless to say, I was immediately sold. I was drinking beer when I saw it so I told my friend about it and asked him to hold my beer. Honestly, the idea might promote something anti-social but I just thought it would be a good exercise to flex my android app and side project skills. I also think that it's somewhat of an appropriate app idea especially for people of Seattle. Have you heard of the Seattle freeze?

Anyways, I fully intend to finish this and release it on iOS and Google Play store if I can find a solid designer who can finish the UI for me. I will get the rest done. So if you like to build cool things and have some experience with Android/iOS design, please inquire within: avcitamer94@gmail.com

I already have the perfect cover picture idea for the app. I just need to email the artist to see if he would be okay with me using it - or see what he would want in return. Check him out: https://www.willmcphail.com/about I'm a big fan of his New Yorker cartoons.
<img src="https://raw.githubusercontent.com/tameravci/tameravci.github.io/master/_posts/EP11Bo5XkAAa7wR.jpg" width="450" height="450">

Without further due, let's get right into it. First things first. Pick a playlist. Honestly, if I'm coding something exciting but also kinda edgy, I'll go with the Social network soundtrack. Enjoy: https://open.spotify.com/album/1ijkFiMeHopKkHyvQCWxUa?si=3SGegSNkTom9JHvelWcyQg. If I'm more in the melodic/dark techno mood, then I'll go with Mind Against or Tale of Us. Pick your poison basically.

Next is obviously alcohol. But I will not take my beer back from my friend. Instead, let's get funky. Pour some kahlua with club soda. And lots of ice. 

Alright, ladies and gentleman, we are ready to code... is what a junior engineer would say. First, we need to design the interface, use cases and the data model. The rest, the implementation, we can just hand-off to a junior engineer. Just kidding. 

Let's start with our requirements aka the use cases. There might be a slight difference between these terms but it's ok.

### Requirements

```
1. Authenticate the user of the application
2. Secretly enter the plans that you wish you could cancel
3. View all the (confirmed and otherwise) plans that you entered
4. Be nofitied when the cancellation desire happens to be mutual! 
     -- Bonus: Celebrate the cancellation of your plans with confetti.
```

That's it. We got to keep it simple because, you know, I have a day job. Also starting simple and getting feedback is perfectly fine. Worse is better. Reduce the scope. Feature down. Underdo your competition. *Winks at 37signals, you guys know it.

Alrighty, so now that we know our requirements, let's make them crystal clear from the technical standpoint. This is where the engineering team usually chimes in. We take the "more abstract" requirements and turn them into tangible technical capabilities.

Security is a huge thing and being able to authenticate the users is essential for almost all applications. We can't take many shortcuts here. However, I will go with the simplest approach here. Normally, I dislike that we still use SIM cards and phone numbers but they do simplify things. I hate social-media app based auth models even more so I'll go with phone numbers.

Obviously, the right thing would be to interact with the UICC SIM card, maybe retrieve IMEI etc, authenticate via some JavaCard applet (https://nelenkov.blogspot.com/2013/09/using-sim-card-as-secure-element.html) but ain't nobody got time for that. Funny, I actually have a solid amount of expertise in JavaCard development but that's for a later post perhaps. We will do the second most feasible approach. One-time OPT codes sent via SMS. Here's the API we are going to build.

```
Given a phone number, generate a one-time use time-based OTP, send it via SMS and ask the user to enter the OTP on the screen
```

or in API form
``` java
/*
Generates an OTP, sends it over SMS, stores it with a TTL for a phone number auth attempt
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

Assuming we can authenticate users, we can now focus on the core use case. Here, it is extremely critical to agree on the time of the dates. I like to go with the granularity level of daily plans, nothing more or less granular. I mean, who makes a plan with the same person more than once on any given day? Our app will allow you to secretly enter your plans on a daily basis and with however many people.

This means, our date consists of 2 questions: 1) With whom (contacts prompt) 2) When? (calendar prompt). Simple. I guess our model does extend to dates with more than just two people. But for simplicity I'll keep it at two for now. I'll try to pay attention to that to avoid making a one-way door decision that would restrict our ability to support dates with multiple people. In such a case, I presume we would need to wait for all parties' cancellation request before the plans can be considered cancelled.

```java
/*
Given a date and a contact, stores the plans that the user wishes they could cancel
*/
void schedule(ClientId authenticatedAndAuthorizedUser, Date when, PhoneNumber withWhom);
```

It's important to note that all these APIs will have to carry the authenticated user id so that the backend can check whether or not this user is authorized to set a date on behalf of the owner of the client id. Usually, those client ids or auth tokens should be short-lived and periodically refreshed. Ignoring that for the time being.

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

We need to be able to send notifications only when both parties cancel on each other. I love the confetti idea. So what I'm thinking is when the feelings are mutual, we should send an SMS notification with a link to open the app, and then boom, the app shows you what plans are cancelled, when & with whom etc and throw a bunch of confetti on the screen. Animations would be dope. We could technically provide an option to revive the date? I'm just thinking out loud.

```java
/*
Given a notification token retrieved via SMS and the authorized user, retrieves the cancelled plan details to view.
*/
Plan getCancelledPlan(ClientId authenticatedAndAuthorizedUser, NotificationToken token);
```

We all love push notifications but you know that's more work. Here we will take advantage of the SMS integration and will force the app to make a backend call to retrieve just the cancelled event. Similarly, we need another API to retrieve all the notifications so that if they miss the SMS or something, they can go back to the app and hit on the notification page and go from there.

One thing that's critical is to NOT send the notification immediately AFTER the second person puts in the plan details. That would make it obvious to both parties who entered the details first. Which is why we should be careful building that core logic. We could introduce some jizzer between the time plans are mutually confirmed and the time notifications are sent out. The exact jizzer should depend on the time that's left until the date. Nothing difficult, meh, implementation detail even but the idea is essential to success. More importantly, we need to be able to convey this feature to our users so that they don't assume the plans are always wished to be cancelled by the other party. Because that's how dating apps work unlike this app.

A couple of geeky ideas I have on this is to show users "secretly" that somebody is wishing to bail on them by sending them a notification without revealing who and when. That way we can lure them into buying the "premium" version of the app that can reveal the identity of the person that's bailing. Maybe. Haven't thought deeply about this so let's get back to our non-existing free version.

Basically, we are done with the APIs. I think I covered all the requirements. Now, assuming these are my right access patterns, I can go ahead and design my data schema/model/state management. Let's talk about state.

### State Management

We need to be able to persist the auth token/client id somewhere on the device that can only be modified by the app. I was thinking of using SharedPreferences in Android world. Once the authentication is complete and the backend sends the client id back to the app, this is where the app will store it. It will be durable across app crashes/device reboots. The app will keep this state until the authentication needs to be repeated due to access expiry. This is pretty much the only state that the app needs to keep track of. Obviously, it can do some caching to minimize redundant calls to the backend.

On the cloud side, we need to keep track of all the entered unconfirmed plans so we can match them. We need durable storage. To make the app fast enough, we will need to define our keys and indices well. We should go with a database - a NoSQL should do it. Good thing, we have a solid idea of our access patterns. Otherwise, I'd recommend going with a SQL approach and just storing the entities per table. A relational model would keep the flexibility in the queries you are going to construct as you develop the idea.

I want to have the fewest number of tables that we can get away it. We should also strive to make the runtime of our queries linear. I came up with the following:

```
Table: Plan
Primary (hash) key: Number1_Number2_Date (Number1_Number_2_..._NumberN_Date to support dates with multiple people)
Sort (range) key: Number that entered the plan details first
```

This way both our write & read (match) operations will be O(1). Here's how it works:

1. Let's say a User 2 with Number 2 enters the plan they wish to cancel with User 1 with Number 1 on Date X.
2. Backend sorts Number 1 and Number 2 numerically and concats them with the date. Let's say it turned out to be Number1_Number2_DateX.
3. Backend first queries the DB with that being the hash key, PK=Number1_Number2_DateX. It finds out there are no records.
4. Backend writes the record into the DB with PK=Number1_Number2_DateX and SK=Number2
5. A day after User 1 with Number 1 also realizes they wish to cancel so they submit the request on BAIL APP.
6. Backend sorts Number 1 and Number 2 numerically and concats them with the date. It becomes Number1_Number2_DateX.
7. Backend queries the DB with that being the hash key, PK=Number1_Number2_DateX. It finds a record in O(1) constant time because that was our hash key. This is good news- confetti time.
8. Backend sets up the notication event and time for further processing.

Unfortunately, we can't query this table for retrieving all the plans given a number so we would have to store the details in a parallel table with PK=Number, and sort keys being withWhom and date and status. Actually mirroring the plan in-memory data structure. That should work and retrieve the plans (sorted by today onwards) in linear time given a phone number (user)

That wraps up the data model. I shall now briefly cover the dependencies that we will utilize and proceed to the implementation in the next blog post.

### Infrastructure and Dependencies

Since I'm an AWS geek I won't look elsewhere. We need a persistent NoSQL data storage and we have no better option than DynamoDB. Simple, consistent API. Easy enough.

We will use SNS with SMS integration. SMS is not inherently "reliable" unless we use Transaction feature supported by SNS. I think OPT may fall into this category. I will have to double check. We will also use SNS for notification texts and they will have to be promotional. Cost should be minimal at first but with more users SMS would cost the company money so we would have to look into actual push notification integration.

CW Events can be used in a fairly straightforward way to integrate "cancel notification" scheduling which would then trigger SNS-SMS push notifications. Once the storage returns a record to step #7, the backend should schedule a CW event.

We should be able to build the entire backend in Lambda. No need for any persistence or customization in the compute layer. Nothing complicated. Short, plain simple functions with no timeout concerns. The cold start would be annoying but now they have reserved dedicated lambdas that we can take advantage of. At first, Lambda would be great since it will scale with usage. If the app became extremely popular, we may have to switch to dedicated servers. Very hypothetical.

Finally, we need to write some APIs. Yeah, yeah, we may need an API Gw but what was the first rule? Simplicity. So we will ignore the GW at first. Instead, we will setup an unauthenticated Cognito pool (since we are doing authentication out of band) and that pool shall give the android app's aws sdk permissions to call our Lambdas. To keep things warm, we can write all the APIs in one lambda or 1 per API. Not so sure yet, implementation detail so I'll cover this in the next post.

Getting excited to build this. Hope you enjoyed this guide around system design and keeping things simple as you build your next idea.
