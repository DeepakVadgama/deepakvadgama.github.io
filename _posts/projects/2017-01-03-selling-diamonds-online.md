---
layout: post
title: Selling Diamonds online
category: projects
comments: true
published: true
excerpt: E-commerce website of diamond trading company Sejal Gems
---

For all the things you can buy online, I haven't seen many sites selling raw diamonds, though there are many selling jewellery. 

Me and my friend Bhavik were tasked with creating such twin websites. 
One site for selling certified diamonds. 
Each such diamond is certified by central authorities, and is expensive. 
Second site for selling parcel diamonds (aka loose diamonds). 
These diamonds are tiny, relatively cheap, bought in bulk and are typically used for creating jewellery. 

<figure>
    <a href="{{ site.url }}/images/blog/sejalgems/sejalgems-home-1.png"><img src="{{ site.url }}/images/blog/sejalgems/sejalgems-home-1.png"></a>
</figure>

<figure>
    <a href="{{ site.url }}/images/blog/sejalgems/sejalgems-twin-site.png"><img src="{{ site.url }}/images/blog/sejalgems/sejalgems-twin-site.png"></a>
</figure>

### Site links

- [Ceritified Diamonds](http://104.199.188.85/#/)
- [Parcel Diamonds](http://104.199.180.127/#/)

### Stack

- Spring MVC 
- AngularJS 1.x
- Bitbucket
- Google Compute Engine

### Technology decisions

- **Hibernate vs JDBC**: Being from Java background I was all for using Hibernate. 
But my partner is from database background and is more comfortable with native SQL queries. 
Thus we settled for using Spring JDBC. It did involve writing lot of code and fixing issues, but that was expected. 

- **Security**: I attempted out-of-the-box Spring solution for security, but was unable to get it running. 
(for a subsequent project, I was able to successfully use it with Spring Boot). Thus, I had to 
create own APIs for login/register.
 
    + BCrypt to encode passwords on server 
    + UUID to generate tokens
    + AngularJS's interceptors to add X-AUTH-TOKEN token to every request
    + JS local-storage to save the token
    + Expiry embedded in token to handle session logout
    + Spring's HandlerInterceptorAdapter to verify X-AUTH-TOKEN for every request

{% highlight javascript %}

// AuthInterceptor
return {
    // Add authorization token to headers
    request: function (config) {
        config.headers = config.headers || {};
        var token = localStorageService.get('token');
        if (token && token.expires && token.expires > new Date().getTime()) {
            config.headers['x-auth-token'] = token.token;
        }
        return config;
    }
};

// Token expiry in service
hasValidToken: 
function () {
    var token = this.getToken();
    return token && token.expires && token.expires > new Date().getTime();
}

// Server token expiry interceptor
return {
    responseError: function (response) {
        if (response.status === 401) {
            localStorageService.remove('token');
            localStorageService.remove('username');
            alert('Login expired. Please login again');
            $location.path('/login');
        }
        return $q.reject(response);
    }
};
            
{% endhighlight %}

- **Template for UI** We opted to buy a [template](https://themeforest.net/item/canvas-the-multipurpose-html5-template/9228123)
 to make the site look pretty. 
This was probably our biggest mistake of the project. Granted, all the static pages look great
 with little effort, but when it comes to JavaScript based functionalities, it was painful to 
 find and resolve issues as listed below. Better alternatives would have been to either buy a 
 Angular based template or create site using plain old CSS.

- **Template issue #1: JavaScript code size** Default, out-of-the-box code for the template contains hundreds of files, all of which is imported. Unlike separate HTML files, all the JavaScript related code was in single JS file of 5000 lines of code!! 
The functions within were so tightly integrated that it took me couple of days to remove unused code. 

- **Template issue #2: JQuery/JS w/ Angular** The template was pure JS, JQuery one. Thus all the function calls 
were based on assumption that page is already rendered by the time function is called. 
 This didnt bode well for our Angular app which renders based on digest/eval [loop](https://www.sitepoint.com/understanding-angulars-apply-digest/). I had to find those functions and make them trigger after initial render.

{% highlight javascript %}
$scope.$on('ngRenderComplete', function (ngRepeatFinishedEvent) {
    SEMICOLON.documentOnLoad.init();
    SEMICOLON.documentOnReady.init();
});

$rootScope.$on('$stateChangeSuccess', function (event, currentState, nextState) {
    $timeout(function () {
        window.scrollTo(0, 0);
    });
});

$rootScope.$on('$stateChangeStart', function (event, toState, toParams, fromState, fromParams) {
    if (fromState.name === 'app.home' && toState.name !== 'app.home') {
        jQuery('.tp-banner').revkill();
    }
});
{% endhighlight %}

- **OCZ Lazy Load** Lazy load of JavaScript using [oc.lazy-load](https://github.com/ocombe/ocLazyLoad) files which came in very handy with many pages of the app. 

{% highlight javascript %}
.state('app.login', {
    url: '/login',
    templateUrl: 'tpl/login.html',
    controller: 'LoginController',
    resolve: load(['js/controllers/login.js']) // lazy load
})
{% endhighlight %}

- **Grunt** This was quite painful since I created the GruntFile myself. Grunt is infamous for this very complexity. 
I spent a week to get the below listed functions working. Most examples on StackOverflow/Internet are 
using different versions or Express or NPM etc. 
Since JS world is moving at a breakneck pace it was not an option to learn Gulp or Browserify or something else. 

  + Adding Proxy for tomcat
  + Building (filerev, uglify, minify etc)
  + Server
  + Live Reload

- **Angular Datatables** [This library](l-lin.github.io/angular-datatables/) is a wrapper on famous [JS only datatables](https://datatables.net/). 
It sufficed to provide sorting, searching, pagination, fixed header, scrolling etc.  
 
<figure>
 <a href="{{ site.url }}/images/blog/sejalgems/sejalgems-search-2.png"><img src="{{ site.url }}/images/blog/sejalgems/sejalgems-search-2.png"></a>
</figure>

- **Dynamic Reports** [This library](www.dynamicreports.org/) is a wrapper over open source version of Jasper Reports. It helped us [create invoices](http://www.dynamicreports.org/examples/invoice) in PDF format.

<figure>
 <a href="{{ site.url }}/images/blog/sejalgems/sejalgems-invoice.png"><img src="{{ site.url }}/images/blog/sejalgems/sejalgems-invoice.png"></a>
</figure>

- **Apache POI** [This library](https://poi.apache.org/) helped us create excel based reports. There were numerous reports required for things like customers, orders, inventory, average pricing etc. 

- **Payment Gateway integration** Writeup coming soon. Yet to implement. 

### List of functionalities

- Login/Register
- Home page for branding
- Search/Results/Download-to-excel
- Cart
- Checkout/Enquiry
- Invoice
- Tracking delivery
- Excel based Reports
- Admin (to manage inventory, users/dealers, prices, delivery etc)
- Misc (Google maps, general enquiry etc) 

### Business decisions

- **Delay for payment gateway** The website creation started in first half of 2015, with UAT deployment in June. The client 
wanted full payment gateway integration before we could make it live. 
We suggested making the site live without it to start getting early feedback but client insisted on having it.
The decision making and paperwork process took whole 6 months before we could resume working on the site again. 

- **Feedback** This same delay also resulted a lengthy list of changes as feedback from the client. 
I think we as developers have responsibility of helping customers understand the importance of Agile and its benefits. 
Failing this, it becomes troublesome for both the parties involved. 

### Learnings

- After creating a subsequent project (post coming soon) completely based on Spring Boot and Hibernate, I regret 
not using full power of Spring Boot in this project. I had to spend lot of time developing functionalities and fixing issues. 
- I also regret using template instead of creating site from basic components. It saved time initially, but 
in the long run, was too time consuming. 
- On positive note, I learned lot of new things like OC.Lazyload, diamond industry workings, Spring filters, Angular digest cycle etc. 