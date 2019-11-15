Title: Static Site Generation with Pelican and Netlify
Date: 2019-11-14 15:00
Category: Web Development
Tags: Python, Pelican, Netlify, Static Sites, #100DaysOfWeb in Python
Slug: static-sites-with-pelican-and-netlify
Authors: Tibor
Summary: This blog is running as a static site generated with Pelican and Netlify. Here is how it is set up. 
Draft: False

# Creating Static Websites with Pelican and Netlify

## Static websites
* Series of webpages with fixed content
* The content does not change, unless something is changed on the backend. 
* The site shows the same content for every user. Users can not interact with the site (create content). 
* The opposite of static are dynamic sites where the users can generate the content, then facebook or twitter etc. 
* Static site are great for blogs, because the users only read, while the admin generate the content. 
* Static website generator 
	* Takes some Markdown content / backend schematics and converts it into HTML
	* Source is saved in Markdown format. 
	* E.g. Pelican 
	* Converts from markdown and converts it to HTML. 
	* This is saved in a different folder. 
	* HTML  static pages are very light weight and fast. 

## Creating a static page
* Create a new repo on Github
* Clone the repo to the disk 
* Into the Repo 
* Create and activate `venv`
* `pip install pelican markdown`
* `pip list`
* `pip freeze > requirements.txt`
* Start form a skeleton website, just run  `pelican-quickstart`. This will go through a little command line dialog. 
	* No need to specify the url yet
	* Pagination is probably helpful, because if there are going to be many articles the list might get long.
	* Yes to the task.py/Makefile
	* All the services: no
* This creates a couple of files and directories
* Pelican will generate the pages from what is saved inside the `content` directory and dump that into the output folder.
* Into the content folder and create the first markdown file.
* The markdown file needs some markers for Pelican to know what to do. 
	* `Title: Something`
	* `Date: 2019-11-14 12:00`
	* `Category: Python`
	* `Tags: 100DaysOfWeb, Python, Static Sites`
	* `Slug: first-post` (short descriptor for the URL)
	* `Authors: Me`
	* `Summary: This is a summary of the first post for the pelican blog. This is an example for the 100DaysOfWeb course.`
	* `Draft: True` — This will prevent showing it on the main page but will only be reachable with the URL `.../draft/first-post`
* The rest of the page should be normal markdown. Just add some example content. I might use this note ^^ 
* To make pelican generate the output just run it  `pelican content`. This will dump it to the output folder. 
* There will be sub-folders for tags and categories. For each of the ones used in the header of the markdown will have a separate html file listing all the articles that define them. Once there are more articles the lists will grow and there will be more files in the sub-folders. 
* Also, there will be an `index.html` in there as a landing page. 
* Also , there should be the first article saved as `first-article.html`.
* In the output folder run `python -m http.server` to start serving the site from `index.html`. This is to test the site locally. Hit localhost:8000/  to see the served page.
* The initial styling… is not so awesome. But I am sure that can be changed. There are a bunch of different themes that you can pick from. I even think you could just create your own. That is not part of the videos. I could dig into this myself.
* Categories are listed in a header. This is why categories should be used more sparingly. Tags can be used more freely.

## Deploying the static site
* Push the code to the repo on GitHub. The files are now hosted on GitHub. 
* How can we now make them available online? Enter Netlify. Netlify is a service to make static sites available online.
* Sign-up and login.
* Connect Netlify with GitHub. 
* “New Site from Git” > Github.
* Select repo. 
* Build settings. We ran the Pelican command locally. But we don’t want to do that for the served site on every change. So we need to tell Netlify the build command. This is just `pelican content`. `content` is the folder containing the markdown files (this could be any other directory).
* And then Netlify needs to know from which directory to serve the page. That would be the `output` directory. 
* > Deploy Site
* This will give you a URL after a few seconds. This can now be reached online. That is it. That’s how easy you can get your blog online. 
* Changes in your Github repo will automatically be published by Netlify. The monitor you GitHub repo and automatically publish the changes.  

## Adding Static Content
* Let’s try something a little more complex. Like adding an image. 
* Images need to go into an `content/images`.  Add any image in there. 
* Just go ahead and add the image to the first post
* `![Alternate Text]({static}/images/logo.png)`. 
* `{static}` tells Pelican to check for the resource in the static file source, which is the `content` folder.  
* That is it. 
* Right now this is not converted to HTML yet. The site will not yet show it. 
* To get this into HTML we need to run the build command again `pelican content` (run in the repo source directory). 

![lpld.io Logo]({static}/images/logo.svg)

* To get this online, push the changes to GitHub. 
* Netlify will automatically process the changes and publish the changes and make them available online. This takes just a few seconds.

## Adding Static Pages
* One Page that will always stay the same every time people visit the page. Not like the blog posts. Something like the `about` page. 
* Similar to images there can be a `pages` folder in the `content` folder. 
* Create that an add a `about.md` file. 
* As a static page, this only needs a `Title:` header for Pelican.
* Just add some more content to the page. 

## Custom Domain
* In the Netlify settings you may also add a custom domain and add a certificate for https. 
