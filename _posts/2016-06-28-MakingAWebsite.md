---
layout: post
title: Making a Website with Jekyll
comments: true
---

How to create a website like this blog with Jekyll.

<!--more-->

Welcome to my new data science blog!  I'll be posting tutorials which explain various data science topics that I come accross in my studies.  My hope is that these posts will provide a valuable resource for aspiring data scientists who are learning similar topics as they start out. 

I thought it would be appropriate for my first post to explain how I created this website.  There are many options on the web to help you build a blog.  The most popular among these include [WordPress](https://wordpress.com/create/), [Pelican](http://blog.getpelican.com/), [Ghost](https://ghost.org/), and [Jekyll](https://jekyllrb.com/).  There are arguments for and against each of these platforms, and the choice of which to use comes down to personal preference and needs. In the end, I decided to create my blog using Jekyll.  

In this post, we'll discuss the following topics:

* [Introduction to Jekyll](#intro)
* [Installing Jekyll](#install)
* [Initial setup of a Jekyll website](#initSetup)
* [Customizing your Jekyll website](#customize)
* [Ideas for Customizing your Page](#ideas)
* [Creating your own domain name](#domain)


## <a name="intro"></a>Introduction to Jekyll

Jekyll is a static website generator which is designed to give complete control of a website's layout through simple HTML and Markdown files.  As a static website generator, Jekyll reads HTML and Markdown files to produce identical content for every visitor to the site.  In contrast, a dynamic website generator such as [Pico](http://picocms.org/) can customize content for each user by interfacing with a Content Management System (CMS) database.  These database queries slow down interactions with the website, and introduce additional complexities which are not needed for a simple blog.  A Jekyll website can be hosted and version-controlled for free on [Github](https://github.com/).  For more information, you can read [this blog post](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html) by Jekyll's creator, Tom Preston.

## <a name="install"></a> Installing Jekyll

There are a number of [prerequisites](https://jekyllrb.com/docs/installation/) that have to be met before Jekyll can be installed on your computer.  You'll need the following items: 
 
 * Linux, Unix, or Mac OS X
 * Xcode
 * Ruby
 * Jekyll

If you are not using one of the operating systems mentioned above, you will need to look for Windows-compatible alternative such as [Pelican](http://blog.getpelican.com/) or [Ghost](https://ghost.org/).  In this section, I'll be describing the Mac OS X installation process.  I haven't installed Jekyll on a Linux environment yet, but I imagine the details of each step are similar.

1. If you're a mac user, you most likely have Xcode installed already.  You can check if this is the case by typing `xcode-select -v` into your terminal.  If the terminal suggests you don't have Xcode installed, you will need to download it from the App Store.  

2. Once Xcode is installed, you will need to install the Xcode command line tools by typing `xcode-select --install` into the command line.

3. Jekyll 3 requires Ruby v2.0.0 or higher, and Jekyll 2 requires Ruby v1.9.3 or higher.  If your mac shipped with OS X, chances are you already have Ruby v2.0.0 installed.  You can verify this by typing `ruby -v` into your terminal.  If you need to install or update your version of Ruby you can follow the [guide on the Ruby website](https://www.ruby-lang.org/en/documentation/installation/).

4. Once an appropriate version of Ruby is installed, we can install Jekyll using Ruby gems by typing `sudo gem install jekyll` into the terminal.  This will prompt the installation of Jekyll and several other gems that it depends on.

That's it!  Your computer should now be ready to build a Jekyll website.

## <a name="initSetup"></a> Initial setup of a Jekyll website

We'll be hosting our Jekyll website on Github.  Before setting up the website, you'll need to follow [Github's guide](https://pages.github.com/) to create your own username.github.io repository (where username is your Github username).  If you aren't familiar with Github, pay close attention to the step four, where the guide explains how to push your index.html file to your online Github repository.  You'll be repeating this process each time you make a change to your website. 

Once you have a username.github.io repository set up, it's time to choose a theme for your website.  Look through the list of pre-built [Jekyll themes](http://jekyllthemes.org/) and decide which you like best.  I used a mix of Lanyon and Hyde to make this website. Once you've made your choice, click on the theme, then use the download button to download a .zip file containing the initial HTML and Markdown files for that theme.  The .zip file should contain a master folder which has subfolders such as **\_includes** and **\_posts** (among others).  Extract the contents of that master folder to your local username.github.io repository.  Note that you will likely overwrite the index.html file that the Github guide had you make when you do so, which is fine.

Congratulations, you're finally ready to build the website!  Open a terminal and navigate to your local username.github.io repository.  Type `jekyll build` into the terminal.  This will combine all of the files in the repository into a new **\_site** folder which we'll use to serve the website.  It's possible you will run into some red text if something goes wrong.  I can't predict the issues you will run into, but a quick google search of the red text typically turns up an easy solution.  If the google search doesn't work, you can look through the theme's github page to try to solve the problem.  You can usually find the theme's Github page by clicking on "Homepage" when looking at the [Jekyll theme's website](http://jekyllthemes.org/).  On the Github page, click on the "Issues" tab at the top and use the search bar to look for posts containing "is:issue" followed by the red text you encountered.  You will need to fix all of the problems and rerun the `jekyll build` command until it passes with no errors.

After successfully building the **\_site** folder, you can view a local version of your website by typing `jekyll serve` into your terminal.  Afterwards, direct a web browser to [http://localhost:4000/](http://localhost:4000) to see your website!  

In order to publish the local version of your website to the internet you need to push the changes in your local repository to your online Github repository.  Doing so requires you to run three terminal commands from your local repository:

1. `git add --all`
2. `git commit -m "Initial Theme Setup"`
3. `git push -u origin master`

Your website should now be accessible on the internet by  pointing a web browser to [username.github.io](username.github.io) (where username is your github username).

## <a name="customize"></a> Customizing your Jekyll Website

In this section we'll take a closer look at the folders you extracted from the theme .zip file.  The theme should have included the following folders and files:

* **\_config.yml**
* **\_includes**
* **\_layouts**
* **public** (this may not exist in your theme)
* **\_posts**
* **index.html**


The **\_config.yml** file is used by Jekyll to configure site-wide parameters.  Open the default **\_config.yml** file provided by your theme in a text editor of your choice.  There should be a number of site-wide parameters that are assigned to generic settings.  Review the contents of the file and determine which parameters should be replaced with your personal information -- for instance, there are probably title, url, and contact parameters that needs to be changed.  If you want to add site-wide parameters that do not already exist, or if you want more information on the **\_config.yml** file, see [Jekyll's configuration page](https://jekyllrb.com/docs/configuration/).

The **\_includes** folder contains HTML files that are later included by the <code>{{ "{% include " }}%}</code> command in other HTML files.  These files are used to produce sections of your website that are appear in multiple places, such as a comments section or a sidebar.  There should also be a **head.html** file in this folder, which will be used for the HTML head of each page on your website. 

The **\_layouts** folder contains a number of HTML files that are used to style different pages on your website.  There is typically a **default.html** file that is used to style the homepage, and a **posts.html** file that is used to style the individual blog posts.  Layouts are assigned to each section of your website using [YAML front matter](https://jekyllrb.com/docs/frontmatter/).  You will see instances of YAML front matter defined at the top of some of the HTML and Markdown files that were included with your theme, with the YAML front matter defined between two series of three dashes such as in the example shown below.

```
---
layout: default
---
```

It's likely that there is also a **public** folder in the main directory that contains some css files.  These css files will provide additional formatting for the layouts' HTML code by using the `<link rel="stylesheet" href=css_file.css \>` syntax within **head.html**.  If you want to change how your website looks, you should begin by editing the HTML code in the **\_layouts** folder and the css files that link into **head.html**.  If you are unfamiliar with HTML and css, Code Academy has a [fantastic introduction](https://www.codecademy.com/learn/web) that will help you learn how to edit these files.    

The **\_posts** folder is where you will store the Markdown files that create your individual blog posts.  Markdown is a plain text formatting language that is designed to be easy-to-read.  You can edit Markdown files with any text editor, but I suggest you use the [MacDown](http://macdown.uranusjr.com/) editor for this purpose.  MacDown will render your Markdown files in real time, and includes a great tutorial explaining Markdown syntax.  To create a new post, simply add a Markdown file to the **\_posts** directory and push the changes to your online github repository.  To ensure the blog posts render correctly, don't forget to assign `layout: posts` (or whichever layout you want to use) in the YAML front matter at the top of the Markdown file. 

Finally, the **index.html** file is used to create the homepage for your website.  If you want to edit the look of your homepage you will need to edit this file.  Be sure to think about any **\_layouts** files, css files, or **\_includes** files that might be interacting with **index.html** when editing it.

## <a name="ideas"></a> Ideas for Customizing your Page

To give you an idea of where to start when customizing your website, I've provided a list of some of the initial changes I made to this website's theme:

* Updated my **_config.yml** file to contain my personal information, and to fix some build issues that I ran into
* Customized my sidebar by combining and editting code from the Hyde and Lanyon .css and HTML files
* Edited my **index.html** file change the layout of the homepage.
* Added excerpt seperators to enable summaries of each blog post.  You can do this by adding an excerpt_seperator to your **\_config.yml** file, including that separator between the summary and the content in your blog post Markdown files, and displaying only the `post.excerpt` content in **index.html**.  See [Jekyll's information](https://jekyllrb.com/docs/posts/) on post excerpts for more details. 
* Added a comments section to my **\_includes** folder by following [Perfectly Random's guide](http://www.perfectlyrandom.org/2014/06/29/adding-disqus-to-your-jekyll-powered-github-pages/)
* Added google analytics to my **\_includes** folder by following [Michael Lee's guide](https://michaelsoolee.com/google-analytics-jekyll/)
* Added social media links to my **\_incldues** folder by following [Aleksandar Todorovic's guide](https://blog.r3bl.me/en/simple-social-media-links-jekyll/)
* Added [Font Awesome](http://fontawesome.io/) icon links to my sidebar by following their [get-started guide](http://fontawesome.io/get-started/)
* Customized the favicon that is displayed in web browser tabs
* Created a subscriber e-mail list using [MailChimp](http://mailchimp.com/) and [WebJeda's guide](https://blog.webjeda.com/jekyll-subscribe-form/)

If you are looking for more details on how I implemented these changes, feel free to look through my website's [source code](https://github.com/Raknoche/Raknoche.github.io).

## <a name="domain"></a> Creating your own domain name

If you've been following this post up until now, you can 
access your website by pointing your browser to [username.github.io](username.github.io). Some people may prefer to have a more professional or unique web address.  If this is the case, you will need to purchase a domain name from a registrar. 

There are many domain name registrars to choose from.  One of the most well known registrars is [Go Daddy](godaddy.com), while some of the best user reviewed registars include [Name Cheap](namecheap.com) and [Gandi](gandi.net).  I chose to use Name Cheap for my website domain due to their pricing, positive user reviews, and easily accessable tutorials that explain how to interact with the registrar's services.  

If you purchase a domain name from a registrar, the registrar is required to display your personal information in a database called [WhoIs](https://www.whois.net/).  Unfortunately, this allows your information to be collected by web scrapers, resulting in unsolicted emails, junk mail, and phone calls.  As a result many registrars offer identity protection services.  Be wary when you purchase these services; they work by replacing your contact information with the contact information of the protection agency, which means the protection agency owns the domain name in the eyes of the law.  This is not as bad as it sounds if you are using a reputable service, but you could easily be taken advantage of if you aren't careful about who is allowed to overwrite your information in the WhoIs database.  In the case of Name Cheap, the protection service is called WhoIsGuard.  I haven’t found any complaints about them stealing domain names, and have found plenty of people saying they are reputable.  Their own [service agreement](https://www.namecheap.com/legal/whoisguard/whoisguard-agreement.aspx) reserves the rights of the domain name to you.  I also found plenty of reports online of people being flooded with emails after opting out of identity protection services.  In light of this, I decided to use WhoIsGuard on my website, but ultimately you'll need to make this decision yourself.

After purchasing your domain name, it might take a few minutes for the change to propagate through the web.  You can check its progress by using [What's My DNS](https://www.whatsmydns.net).  


Once you own a domain name, you can redirect your github.io page to the domain name with a few simple steps.

1. Go to your repository on github: ([https://github.com/username/username.github.io](https://github.com/username/username.github.io))
2. Add a new file to the repository named "CNAME"
3. Put the full address of your domain on the first line of CNAME
4. Commit the CNAME file
5. Go to your domain name registrar and add a CNAME DNS record pointing to your github page.  Name cheap has a [guide](https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages) on how to do this for their service.  The process should be similar for other registrars.

After completing the steps above, you will be able to access your website using your new domain name in addition to [username.github.io](username.github.io).  

Finally, if you'd like an e-mail address that is associated with your new domain name, you can follow [WebJeda's guide](https://blog.webjeda.com/free-domain-email-zoho/#step-1-log-in-to-zoho) that explains how to do so for free by using [Zoho](https://www.zoho.com/).


