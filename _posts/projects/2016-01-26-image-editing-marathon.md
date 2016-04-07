---
layout: post
title: Image editing marathon
category: projects
comments: true
excerpt: Wordpress website for Balaji Extrusions
---

I recently created a Wordpress based website [Balaji Extrusions](http://balajiextrusions.com). It was one of my first projects as a freelancer/consultant.
 
## Wordpress installation mess
It took me 2 days to get Wordpress working on my desktop (Linux Mint). Ironic, considering Wordpress promises installation in [5 minutes](http://codex.wordpress.org/Installing_WordPress#Famous_5-Minute_Install). 
Turns out, the Wordpress available in Ubuntu repositories is outdated and its [guide](https://help.ubuntu.com/community/WordPress) is no help either.
Found his guide by [Digital Ocean](digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-14-04), which is exceptionally crisp and accurate. 
With this, installation was done in less than 15 minutes.

## Wordpress
Wordpress excels in addressing usual suspects of setting up a website. All these were relatively easy to setup.   

+ [Google Analytics](www.wpbeginner.com/beginners-guide/how-to-install-google-analytics-in-wordpress/) 
+ [Caching](https://wordpress.org/plugins/w3-total-cache/)
+ [Free HTTPS]({{ site.url}}/blog/letsencrypt-gcloud-wordpress/)
+ Email - [Plugin](https://wordpress.org/plugins/postman-smtp/) and [Sendgrid on Google Cloud](https://cloud.google.com/compute/docs/tutorials/sending-mail/using-sendgrid) 
+ [Permalinks](https://codex.wordpress.org/Settings_Permalinks_Screen)
+ [Brainstorm Widgets](https://ultimate.brainstormforce.com/)
+ [Visual Composer](https://vc.wpbakery.com/)
+ [Child theme](https://codex.wordpress.org/Child_Themes) to allow easy updates of theme.

## Theme 
Even though the website did not feature a blog, I chose a theme so as to minimize the effort in creating a beautiful looking site.
I chose [Porto theme](http://themeforest.net/item/porto-responsive-wordpress-woocommerce-theme/9207399?s_rank=3) and it was a big learning for me. 
 I had to muck around a lot to resolve simple issues (compounded by my lack of Wordpress experience). 

+ My learning: Choose a theme with a lot of sales like [Avada](http://themeforest.net/item/avada-responsive-multipurpose-theme/2833226?s_rank=1) or [X-Theme](http://themeforest.net/item/x-the-theme/5871901?s_rank=2). 
+ Porto sales = ~4,500, X Theme sales = ~95,000, Avada sales = ~200,000!
+ Higher sales = Good standard documentation
+ Higher sales = Lot of indirect documentation in form of Q&A from other buyers and YouTube videos.
+ Higher sales = Most of the edge case issues have been ironed out.

## Image editing marathon
60% of my time on this project ended up being spent on editing the images. 
Though thanks to that I learned [Gimp](https://www.gimp.org/) a bit and found this amazing tool called [ImageMagick](http://imagemagick.org/).
Few operations/aspects of these tools I learned are:

### Gimp

+ [Image resize](https://www.gimp.org/tutorials/GIMP_Quickies/) and [Canvas select a.ka. Crop](https://docs.gimp.org/en/gimp-image-resize.html)
+ [Transparent background](https://docs.gimp.org/en/plug-in-colortoalpha.html)
+ [Excellent Selection tools](https://docs.gimp.org/en/gimp-tools-selection.html) - By selection, by color, by fuzzy logic its all here.
+ [Layers](https://docs.gimp.org/en/gimp-image-combining.html)
+ [Gradient](https://docs.gimp.org/en/gimp-concepts-gradients.html)
+ [Clone](https://docs.gimp.org/en/gimp-tool-clone.html) and [Erase](https://docs.gimp.org/en/gimp-tool-eraser.html)
+ Export as PNG to retain transparent background. JPEG does not support it. 

### ImageMagick

+ [Bulk Edit](https://www.imagemagick.org/discourse-server/viewtopic.php?t=21454) - Most time saving aspect of this tool. 
+ [Padding](http://imagemagick.org/Usage/thumbnails/#pad)
+ Command line based. 
+ Wicked fast.

## Client communication

+ Clients are very visual. Agile methodology of customer involvement and splitting work in multiple phases is critical. 
+ Its better to coordinate with only 1 point of contact if working for a client company.  
+ Though, as happened with me, chances are, at latter stages once other folks will have feedback and that will have to be incorporated too. 
+ Emailing work to your clients is not enough. 
 You have to be physically be there to walk them through, otherwise the emails are mostly ignored or replied to very late.


## Conclusion
 Project lasted much longer than I anticipated. Most of this time was consumed in image editing. 
  Wordpress & the theme had some learning curve, though very handy once learned. 
 All in all, lot of learning. That makes me happy!