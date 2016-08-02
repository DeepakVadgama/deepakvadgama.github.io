---
layout: post
title: The soul of an Android app
category: projects
comments: true
published: true
excerpt: Android app created for Udacity Nanodegree final project
---

<iframe width="420" height="315" src="https://www.youtube.com/embed/exMCBLn8sCg" frameborder="0" allowfullscreen></iframe>

## Overview
Recently, as part of Udacity's Android Nanodegree course, I created an Android app called Radhe Krishna Bhakti. 
I chose to create this app hoping it will help me in my own spiritual journey and to give it out to [JKP](http://jkp.org). 
They have thousands of followers, so hopefully many will use the app and benefit from it. 
 
It took me nearly 2 months to finish the app, of which, last 3 weeks I was working on it full time. 
This excess time required was due to 2 reasons:

- Restrictions imposed by Udacity on techniques to use.
- Android itself requires lot of boilerplate code to work. 

## App Features
- List of items (Quotes, Stories, Lectures, Kirtans and Pictures)
- Manage Favorites
- Search
- Share
- Feedback
- About

## Backend / REST API
Backend for the app was created using Spring MVC and MySQL. Creating REST APIs in Spring MVC is easy (even better with Spring Boot).
 The app is deployed to Google Cloud (Compute Engine), with Tomcat as the container. I have been working with Compute Engine since 2 years, 
 thus this too was quick. The data was pre-populated in MySQL. The images for content were hosted on Google Storage, while videos 
  were YouTube ones. Overall, this part of the setup went relatively smooth. Links to setup Compute Engine:
  
- [Create standard instance](https://cloud.google.com/compute/docs/instances/create-start-instance) with Ubuntu 16.04 OS, 3.75GB RAM, 1vCPU.  
- [Install JDK](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)
- [Install Tomcat](https://www.digitalocean.com/community/tutorials/how-to-install-apache-tomcat-8-on-ubuntu-16-04)
- [Install MySQL](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-16-04)
- [Add firewall rule for MySQL (3306)](https://cloud.google.com/compute/docs/networking) to allow connections from local machine to run queries (less secure).
- [Add firewall rule for Tomcat (8080)](https://cloud.google.com/compute/docs/networking) or change tomcat config to listen to port 80 instead.
- [Copy](https://cloud.google.com/sdk/gcloud/reference/compute/copy-files) over WAR/JAR file manually to server, or setup Git/Maven on server itself. 

## Environment setup

- Android Studio: Android tooling has progressed leaps and bounds in just 1 year due to Android Studio. Setup is pain free and works beautifully.
- Instant Run: Instant run avoid complete rebuild of project in case of minor code changes.   
- Gradle: Gradle speeds have improved and combined with Android Instant run, development is fun.
- Use latest SDK and build-tools version, and keep minSDK as 16 to target [90% of Android install base](developer.android.com/about/dashboards/index.html)
- Keep minSDK version for dev config as 21 to avoid time consuming dex merge process. 
- [My build.gradle file](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/build.gradle) 

## Content Provider aka The Beast
This part of the app took the most time to implement. This part was mandated by Udacity. 
It involves:

- creating table creation/insert/update/delete queries
- building URIs for all requests expected
- building contracts for table/columns
- creating cursor loader to load data
- creating cursor adapter to use the data loader

This [whole flow of Loader](http://www.androiddesignpatterns.com/2012/07/loaders-and-loadermanager-background.html) is quite detailed and can be [difficult to understand](https://www.udacity.com/course/viewer#!/c-ud853/l-3681658545/m-1593528720). 
I now know why services like Firebase are so useful. They help developers save a lot of time. 

## SyncAdapter 
I used SyncAdapter to schedule synchronization with data. 
Though now, Google recommends using JobScheduler, this part too was mandated by Udacity. 
SyncAdapter's ability to sync immediately or configure periodic sync came handy. I added settings, for user to change duration between syncs.
While, the immediate sync option was useful in implementing 'Pull to refresh' and menu item 'Refresh' to get new data from server. 

## Recycler View / List View
RecyclerView is the shiny and well-thought out (I presume) cousin of ListView. Though, it has quite extensive API. There are 
 dedicated sessions for RecyclerView in [every](https://www.youtube.com/watch?v=imsr8NrIAMs) [Google IO](https://www.youtube.com/watch?v=LqBlYJTfLP4). 
I opted for ListView to implement list of items instead of RecyclerView. 
There is native support for CursorAdapter with ListView but not for RecyclerView.    

## SearchProvider
All of the Google apps come with beautiful looking search bars, which show suggested/recent queries.
I wasn't able to find out-the-box pretty looking search bar for this. 
Related [Google article](https://developer.android.com/guide/.../search/adding-recent-query-suggestions.html) details the steps. 
It works, though I couldn't get it to look as good due to time constraints.  

## Libraries
- Retrofit: Probably a gold standard for REST calls.
- Glide: To download, cache and load image into Views.
- Like button: Animated 'like' button similar to Twitter's. 

## Google Identity
I used [Google's Identity](https://developers.google.com/identity/sign-in/android/start-integrating) support to retrieve user's email id. 
This was used to synchronize favorites with the server and across devices.
The synchronization of existing favorites from server to app happens during account selection. 
Sync from app to server happens in batches using SyncAdapter. 
Google Identity also provides [token based authentication](https://developers.google.com/identity/sign-in/android/backend-auth), which helps in authorizing requests on the server.

## YouTube Embed
- Setup: I found YouTube library to be difficult to setup. 
- Thumbnails: Once setup, I could not get the thumbnails to work for 2 days straight. I gave up and used YouTube's URL which is used to retrieve thumnails. [Code here](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/src/main/java/com/deepakvadgama/radhekrishnabhakti/util/YouTubeUtil.java)
- Videos: Setting up embedded video on the other hand required very little [code](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/src/main/java/com/deepakvadgama/radhekrishnabhakti/DetailFragment.java).  

## Material Design
- [Icons](https://design.google.com/icons/)
- [Palette setup](https://www.materialpalette.com/)
- [Keylines](https://material.google.com/components/cards.html)
- [Font sizes](https://material.google.com/style/typography.html) 
- [Transitions](https://developer.android.com/training/material/animations.html) 

## Notifications and Sharing
Creating notifications and sharing for quote/story (text based) was simple. 
For pictures/lectures/kirtan (YouTube thumbnail), it requires background thread to download. 
I used Glide library with AsyncTask to implement 
both [notifications](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/src/main/java/com/deepakvadgama/radhekrishnabhakti/util/NotificationUtil.java) and [sharing](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/src/main/java/com/deepakvadgama/radhekrishnabhakti/util/SharePicturetask.java). 

## Analytics
Google Analytics [setup and use](https://developers.google.com/analytics/devguides/collection/android/v4/) in Android is surprisingly straight forward. 
It took me only 2 hours to setup analytics for all screens and other events I wanted to track.   

## Widget
Creating a widget was also mandated by Udacity. Thankfully, its simple to [implement](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/src/main/java/com/deepakvadgama/radhekrishnabhakti/widget/QuoteWidgetIntentService.java). 
Extend AppWidgetProvider and use IntentService to get data and bind to widget views. 
I created widget which displays quotes, and configured it to update once every day. 
 
## Accessibility, i18n & Corner cases
- i18n: Externalize all strings in strings.xml instead of hard-coding them. 
- Talk back: Mostly requires adding content-description to all ImageViews, buttons and such. 
- DPad: Requires adding attributes (focusable, descendantFocusability, clickable) in relevant xmls.
- RTL: Requires adding start/end instead of left/right in layout xmls.
- Corner use cases: Address cases like Internet not working, unable to fetch data, no search results found etc, and display [appropriate message on screen](https://github.com/DeepakVadgama/Capstone-Project/blob/master/app/src/main/java/com/deepakvadgama/radhekrishnabhakti/BaseActivity.java#L192). 

## App Signing 
Signing app was mandated by Udacity and was made easy by this [Google article](https://developer.android.com/studio/publish/app-signing.html) detailing the steps. 

## Final Thoughts
All in all, creating app was time-consuming but fun. 
Especially once ContentProvider/Loader/SyncAdapter were implemented, other features were more fun due to quick code-test feedback loop. 
I submitted the project and it was reviewed successfully. 
My next steps would be to polish the app (RecyclerView, Material Design motion, Navigation Drawer) and create Web-Admin to add content dynamically.    

<figure>
 <a href="{{ site.url }}/images/blog/capstone/screenshot1.png"><img src="{{ site.url }}/images/blog/capstone/screenshot1.png"></a>
</figure>

<figure>
 <a href="{{ site.url }}/images/blog/capstone/screenshot2.png"><img src="{{ site.url }}/images/blog/capstone/screenshot2.png"></a>
</figure>