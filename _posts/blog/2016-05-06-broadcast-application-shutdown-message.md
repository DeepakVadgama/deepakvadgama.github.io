---
layout: post
title: Broadcast shutdown/restart messages to all application users
category: blog
comments: true
excerpt: Broadcast message to all users before shutdown and post restart (Spring, JSP and JQuery) 
tags: 
  - spring
  - javascript
---

I have been working on an [ERP software]({{ site.url }}/projects/refactoring-to-sanity/) which is used by ~25 users across departments. 

The coordination across departments during a deploy (before shutdown and post restart) becomes a pain. 
Unfortunately, the company does not use chat tools which can be used for broadcast. 
Instead, the communication is done using phone or in-person.
 
To resolve this, I was asked if software itself could intimate the user when it is about to shutdown, and once its restarted.  

> Do not overengineer

I started looking into websockets and other forms of push communication, though quickly realized it would be an overkill. 
Since there are only 25 users polling becomes the simplest solution. 


## Server side - APIs for checking current status and updating status to 'shutdown'

- API for shutdown can either be based on authentication or a secret token so that its not misused (mine is on a self-contained LAN and not internet, so I used token)    
- API for startup can be added if required. Considering, once deployed, the atomic boolean will be reinitialized automatically.  
- AtomicBoolean is used to avoid caching issues (Volatile can also be used).  
- During status check, return 'ok' for normal process, and 'shutdown' if boolean is false, indicating shutdown will begin shortly.    

<br>
{% highlight java %}
@Controller
@RequestMapping("/meta")
public class MetaController {
	
	private static final Logger log = LoggerFactory.getLogger(MetaController.class);
	
	private AtomicBoolean status = new AtomicBoolean(true);
	
	@RequestMapping(value = "/status", produces = "application/text", method = RequestMethod.GET)
	@ResponseBody
	public String status() {
		if (status.get()) {
			return "ok";
		} else {
			return "shutdown";
		}
	}

	@RequestMapping(value = "/shutdown", produces = "application/json", method = RequestMethod.GET)
	@ResponseBody
	public boolean shutdown(@RequestParam String secret) {
		if ("matrix".equalsIgnoreCase(secret)) {
			status.set(false);
			log.info("Shutdown indicator turned on");
		}
		return true;
	}
}

{% endhighlight%}

## Server side - Permissions

- Ensure the status check URL is open for all (no authentication).   
- This will depend on your session implementation. In our case once server is restarted, the sessions are invalidated (user checking status after server restart will be redirected to login in that case, which we want to avoid).

  <br>
{% highlight xml %}

<http auto-config="true" use-expressions="true">
	<intercept-url pattern="/login*" access="permitAll" />
    <intercept-url pattern="/meta/status" access="permitAll" />  <!-- Add this line -->
    <intercept-url pattern="/**" access="isAuthenticated()" />
    <form-login login-page="/login" default-target-url="/welcome" authentication-failure-url="/loginfailed" />
</http>

{% endhighlight%}


## UI side - Poll for status and alert if shutdown message received

- Use $.ajax call to check for status, and alert user accordingly.  
- Use JQuery's timeout to periodically call the same method. This is done by calling timeout in onComplete callback (avoid calling timeout in success callback, because once server shuts down, the ajax call will fail and next timeout will not be set).  
- Put this code in common JavaScript, JSP, Tag file. I used it in menu.tag file, since all pages include that file for showing header menu.  
- In case, the JQuery library is being loaded *after* this code is called. Use defer (as shown below) to wait for JQuery to load.
- Use modal in case you want the alert to be pretty. I used the native one. 

<br>

{% highlight javascript %}

var software_shutdown = false;
    
// Wait for jquery to load
(function defer() {
    if (window.jQuery)
        worker();
    else
        setTimeout(function() { defer(worker) }, 100);
})();

// Function to check status and alert user
function worker() {
    $.ajax({
        url: 'meta/status',
        success: function (data) {
            if (data === 'shutdown' && software_shutdown === false) { // Check flag to show alert only once
                software_shutdown = true;
                alert('Please complete your work. Software will shutdown in 5 minutes.');
            } else if (data === 'ok' && software_shutdown === true) {
                alert('Software restarted. Please refresh page and resume work.');
                software_shutdown = false;
            }
        },
        error: function () {
            alert('Software unreachable. Please try again in 5 minutes. ');
        },
        // Complete instead of success, because if first call fails, it will never set next timeout.
        complete: function () {
            setTimeout(worker, 60000);
        }
    });
}

{% endhighlight%}


## Test and Confirm 


### Call shutdown using URL and secret token.
<figure>
 <a href="{{ site.url }}/images/blog/broadcast/shutdown.png"><img src="{{ site.url }}/images/blog/broadcast/shutdown.png"></a>
</figure>

### Check if UI receives shutdown message and alerts the user.
<figure>
 <a href="{{ site.url }}/images/blog/broadcast/ui-shutdown.png"><img src="{{ site.url }}/images/blog/broadcast/ui-shutdown.png"></a>
</figure>

### Wait for 5 minutes, restart server and check if UI alerts user of restart.
<figure>
 <a href="{{ site.url }}/images/blog/broadcast/ui-restart.png"><img src="{{ site.url }}/images/blog/broadcast/ui-restart.png"></a>
</figure>

