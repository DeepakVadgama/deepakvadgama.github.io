---
layout: post
title: '[2015] Perils of startup world'
category: projects
published: true
excerpt: My experience working on a failed startup chat2get.com.
---

One fine day in 2013, I received a call from an ex-colleague about an opportunity. It was an invitation to work on a startup. It was exciting to say the least. 

## Idea
The idea was to connect users to small businesses and individual contractors (eg: carpenters, cake shop etc) using a chat based app. The startup was formed by [Harshil Shah](https://www.linkedin.com/in/aharshil) and [Timo Steiz](https://www.linkedin.com/in/timosteitz) (who also runs [shoesize](http://shoesize.me)). They had just finished their MBA in Spain and were planning on this idea to launch in Barcelona. The revenue model was to charge small amount from the businesses on every lead generation. Here is the [presentation/pitch](https://drive.google.com/open?id=0BwwLZCExviQsNHpQcVN4MzRQWlhVR1FuYV95ZmlndnAzYi1n). 

There are now quite a few startups with same idea like [HouseJoy](www.housejoy.in/home_services‎) and [UrbanClap](https://www.urbanclap.com/).  From the other end, popular chat applications like [Facebook Messenger](http://thenextweb.com/facebook/2015/12/02/facebook-now-lets-you-chat-with-businesses-via-messenger-on-their-website/#gref) and [WhatsApp](https://techcrunch.com/2016/08/25/whatsapp-plans-to-let-businesses-on-to-its-service-before-the-end-of-the-year/) have started to onboard businesses, entering the same space. 

## Team 
The team comprised of 2 co-founders Harshil & Timo, and 2 developers Bhavik and myself. We hired few interns for marketing but they were temporary hires working only for a short duration. We (developers) were promised a generous share of the startup at 12.5% a piece. 

## My Role
My role was to create and drive the entire product (which was a mobile optimized web application). I setup the project environment, created the core flows (authentication, db connectivity, etc). After this me and Bhavik split up the work on functionalities like signup, login etc. Post that I created the chat flows and optimized website for mobile. 

## Tech Team
Bhavik is a Database Developer (Oracle) and was not well versed with Java or with AngularJS back then. I helped him learn both, by spending lot of time walking him through the live code. All the effort was completely worth it. He is quite hard working and we complement each other nicely. We still work together on other projects. We communicated almost everyday (hangouts was more than handy) and worked late hours and weekends (we both were working on full-time jobs). This development phase which lasted more than 8 months was the deceptively most exciting.

## Tech stack
- Servlets API on Tomcat (I know, we did not even use Spring).
- MongoDB
- AngularJS (1.x)
- Bootstrap

## Functionalities

### Easy 
Once the Angular and Servlets core flows were set up, implementing these flows was quite easy.

- Static Website
- Login, Signup, Forgot Password - Including for Businesses
- Notification to business about new chat
- Enquiry


<figure>
 <a href="{{ site.url }}/images/blog/chat2get/home.png">
   <img src="{{ site.url }}/images/blog/chat2get/home.png">
 </a>
</figure>

 
### Moderate

- **Search** - Searching within MongoDB using Java services. 
- **Auto complete** - We used Twitter’s [typeahead](https://twitter.github.io/typeahead.js/examples/)
- **Multiple languages** - [3rd party translate library](http://github.com/PascalPrecht/angular-translate). 
- **Auth token** - Used UUID sent from server and [Angular cookies](https://docs.angularjs.org/api/ngCookies/service/$cookies) to save on UI


<figure>
 <a href="{{ site.url }}/images/blog/chat2get/search-results.png">
   <img src="{{ site.url }}/images/blog/chat2get/search-results.png">
 </a>
</figure>

<figure>
 <a href="{{ site.url }}/images/blog/chat2get/home-lang2.png">
   <img src="{{ site.url }}/images/blog/chat2get/home-lang2.png">
 </a>
</figure>


### Hard

- **Chat** - Push messaging was not famous yet, there were limited browser support solutions like [socket.io](). We completely missed [Firebase](https://firebase.google.com). It could have saved us almost 1-2 months. We had to implement messaging using polling (gap of 3 seconds). Two polling channel, one for number of chats (which also handled any new unread messages) and other for latest messages within the active chat.

- **Mobile site** - Bootstrap is quite handy though adding mobile specific css later is quite time consuming.


<figure>
 <a href="{{ site.url }}/images/blog/chat2get/user-chat.png">
   <img src="{{ site.url }}/images/blog/chat2get/user-chat.png">
 </a>
</figure>
<figure>
 <a href="{{ site.url }}/images/blog/chat2get/mobile-combine2.png">
   <img src="{{ site.url }}/images/blog/chat2get/mobile-combine2.png">
 </a>
</figure>


## Startup struggles
The struggle part can be fun, sitting in a dark corner creating the product, fascinating about the millions who would use it eventually. We took more than 8 months to create product and even more months to launch it and try and signup the businesses. 

The tech team had constant friction with the co-founders who wanted the product to be as polished as possible, while we (developers) had been working for close to a year working on the product and wanted to launch it ASAP. 

## Marketing 
It was a chicken and egg problem for the product to take off. Without enough users it was difficult to onboard businesses. Without businesses, product would not be used by customers. We hired few interns to for door-to-door marketing (onboarding businesses) in Barcelona but we couldn’t gather enough. Harshil moved to Hong Kong and we decided to launch the product there but same underlying issues ensured it never took off. 

## Doctive.de - Another opening
We got another opportunity to progress the platform when a german company started by German doctors asked to create a prototype of our product for them. They wanted to use our product to connect Medical Practitioners with Doctors. We charged a reasonable amount and rebranded the app for them. Alas, they never replied back.

<figure>
 <a href="{{ site.url }}/images/blog/chat2get/doctive-home.png">
   <img src="{{ site.url }}/images/blog/chat2get/doctive-home.png">
 </a>
</figure>

## Startup Learnings

As you might have guessed, the startup did not pan out. There wasn’t any traction with the users and we didn’t do much to onboard the businesses. I learned a few important lessons.

- Launch first polish later.
- Fail early.
- Learn to say no to inconsequential changes which demand too much effort.
- Discuss and decide the time lines and targeted milestones. 
- Automate build, deploy, rollback.
- Seek existing (open-source) solutions for complex technical problems.
- Low friction within tech team works wonders.

