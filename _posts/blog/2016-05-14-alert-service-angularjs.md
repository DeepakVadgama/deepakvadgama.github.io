---
layout: post
title: AlertService in AngularJS using Bootstrap
category: blog
comments: true
excerpt: Alert service to reduce boilerplate code for creating alerts throughout the application.
tags: 
  - angularjs
  - javascript
---

I've been working on a web application using AngularJS and bootstrap. 
Most pages have forms that must be validated and user actions to be acknowledged (success/failure). 
[Bootstrap](http://getbootstrap.com/) provides [simple way](http://getbootstrap.com/components/#alerts) to add alerts for achieve this.
     
<figure>
 <a href="{{ site.url }}/images/blog/sejal/login-alert.png"><img src="{{ site.url }}/images/blog/sejal/login-alert.png"></a>
</figure>

Easiest way to add these alerts dynamically (with AngularJS) is to create a boolean flag in $scope, and use ng-if in html to display the alert.

{% highlight html %}

<div ng-if="loginFailed" class="alert alert-danger alert-dismissible" role="alert">
  <button type="button" class="close" data-dismiss="alert" aria-label="Close"></button>
  Failed to sign in!
</div>

{% endhighlight %}

Though very quickly, these number of validations sky rocket (especially for large forms like register page), and adding a boolean 
variable in $scope and creating div tag in html for each validation becomes cumbersome. Also, code becomes too large & lot less readable.

## Solution

All these alerts can be clubbed into AlertService. Subsequently, all controllers throughout the application can add alerts to respective pages
using that service. 

Let's walk through the solution code.

## Step 1: Create AlertService

Create alert service to be used across the application. 

{% highlight javascript %}
angular.module('app')
    .factory('AlertService', ['$rootScope', '$timeout', function ($rootScope, $timeout) {

        // clear alert after routing
        $rootScope.$on('$stateChangeSuccess', function () {
            alertService.clear();
        });
        
        // alerts array on root scope so that all controllers can access it
        $rootScope.alerts = [];
        return {
        
            add: function (type, msg, section) {
                if (section == undefined) section = 1;  //default section number
                $rootScope.alerts.push({
                    'type': type, 'msg': msg, 'section': section, close: function () {
                        return alertService.closeAlert(this);
                    }
                });

                // If you want alerts to disappear automatically after few seconds.
                // $timeout(function(){
                //     alertService.closeAlert(this);
                // }, 5000);
            },
            closeAlert: function (alert) {
                this.closeAlertIdx($rootScope.alerts.indexOf(alert));
            },
            closeAlertIdx: function (index) {
                return $rootScope.alerts.splice(index, 1);
            },
            clear: function () {
                $rootScope.alerts = [];
            }
        };
    }]);
{% endhighlight%}


The alert service retains all the alerts in rootscope so that all controllers can access alerts array. 
It exposes functions add, close and clear for respective actions. 

- Type = type of bootstrap alert - warning, success, danger etc.  
- Msg = message to be displayed to the user  
- Section = section of html page in which you want the alerts to be displayed. More on this later.    

On route change success, clear the alerts so that alerts of one page are not displayed on other. 
The event to listen for is $stateChangeSuccess (since I am using State Router). 
 If you are using older router of AngularJS please use $routeChangeSuccess

Note: Please add the file in index.html script tag. 


## Step 2: HTML code to display alerts

{% highlight html %}

    <!-- section 1 / default -->
    <div ng-repeat="alert in alerts | filter:{section:'1'}" class="alert alert-{{alert.type}}" role="alert" close="alert.close()">
        <button type="button" class="close" data-dismiss="alert" aria-label="Close" ng-click="alert.close()"><span aria-hidden="true">&times;</span> </button> {{alert.msg}}
    </div>
    
    ... some other html content

    <!-- optional section 2 -->
    <div ng-repeat="alert in alerts | filter:{section:'2'}" class="alert alert-{{alert.type}}" role="alert" close="alert.close()">
        <button type="button" class="close" data-dismiss="alert" aria-label="Close" ng-click="alert.close()"><span aria-hidden="true">&times;</span> </button> {{alert.msg}}
    </div>

{% endhighlight%}

This HTML code snippet uses the 'alerts' array from the rootScope, iterates through it using ng-repeat and creates those many alert tags with appropriate class and message. 

Notice the filter pipe of the div tag above. It filters out the alerts only for that specific section. 
Thus if your page has 2 separate sections for alerts, use second div mentioned above. 
In fact you can keep adding sections, but practically 2 should suffice. 

## Step 3: Create alerts from your controllers or services

Add alerts from your controllers/services 
{% highlight javascript %}
app.controller('SearchController',
    ['$scope', '$http', '$state', 'AlertService', 'SearchService',
        function ($scope, $http, $state, AlertService, SearchService) {
            
            // Search
            $scope.search = function () {
                SearchService.search($scope.criteria).then(function (response) {
                    if (response.data == null || response.data.length == 0) {
                        AlertService.add('warning', 'No search results found for the criteria');
                    } else {
                        AlertService.clear();
                    }
                }).catch(function (response) {
                    if (response.data === 'SEARCH_LIMIT_CROSSED') {
                        AlertService.add('warning', 'Too many results found. Please refine your search criteria.');
                    } else {
                        AlertService.add('danger', 'No search results found');
                    }
                });
            };
    ]);
{% endhighlight%}
    
## Step 4: Confirm
<figure>
 <a href="{{ site.url }}/images/blog/sejal/search-results.png"><img src="{{ site.url }}/images/blog/sejal/search-results.png"></a>
</figure>

Thats it. Now in every page you want alerts, just add <alert-service> directive in html and use AlertService in corresponding Controller.

Hit me in the comments if you are stuck implementing this. 