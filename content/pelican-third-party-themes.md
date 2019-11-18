Title: Pelican Third-Party Themes
Date: 2019-11-17 19:00
Category: Web Development
Tags: Python, Pelican, Netlify, Static Sites, Theme
Slug: pelican-third-party-theme
Authors: Tibor
Summary: The current default theme for pelican is not responsive. You could create a totally custom one or add an existing third-party pelican theme to your page. This article explains how to do the latter.
Draft: False

* Current default theme is not responsive. I am currently working to add at least a [responsive grid layout](https://github.com/tbrlpld/pelican/tree/add-mcss-grid). 
* You could create your own theme. Check out the [Creating themes — Pelican 4.2.0 documentation](https://docs.getpelican.com/en/stable/themes.html#) to see how to do this.
* Or, you could just apply an existing theme to your blog. Have a look at this [list of available pelican themes](http://pelicanthemes.com/)

* I’am going to use the [Attila theme](https://github.com/arulrajnet/attila/tree/02dcad911ba1eb2d797a79ec008a810d89a2fde1) . 
* Also, I will make use of the [pelican-themes](https://docs.getpelican.com/en/stable/pelican-themes.html#) command line tool. 

* First thing, go to to the [GitHub page of the Attila theme](https://github.com/arulrajnet/attila/tree/02dcad911ba1eb2d797a79ec008a810d89a2fde1) and download the repo as a zip archive. (Attention though, this is the current development version. If you have issues make sure to checkout the [releases page](https://github.com/arulrajnet/attila/releases) and download the last stable release. Using a stable release is not a guarantee that it will work though. I was getting error messages with the 1.3 version, which is why I switched to the master branch version in which that particular problem was already fixed.)
*  Unzip the downloaded archive
```shell
$ cd ~/Downloads
$ unzip attila-master.zip
```
* This should produce a folder `attila-master/`

* Navigate into your blog projects directory and activate the virtual environment in which pelican is installed.  Because I store my virtual environments in the projects I am using them in, for me that looks like this:
```shell
$ cd ~/dev/blog
$ source venv/bin/activate
```
* Depending how you manage your virtual environments, you might have to use a different command. 

* To show the currently available pelican themes run 
```shell 
$ pelican-themes -l
simple
notmyidea
```
* To install the newly downloaded theme from the unzipped directory and check that it is available by listing the installed themes again. 
```shell
$ pelican-themes -i ~/Downloads/attila-master/
$ pelican-themes -l
attila-master
simple
notmyidea
```

* So far, if you run `pelican content`  your output is rendered with the `notmyidea` theme. 
* To change that, you could use the command line argument `-t`. E.g. running `pelican content -t simple` should render the blog using the simple theme. The simple theme uses no css at all. It renders like plain HTML. 
* To see how the page has been rendered run `make serve` in the blog project main directory (given that you have the `Makefile` set up during the `pelican-quickstart` dialog). If that does not work for you, you may run `python -m http.server -d /output`.    
* If you now visit `localhost:8000` in you web browser you should see the rendered output. 
* To get the output rendered with the newly installed Attila theme, you need to run `pelican -t attila-master content/`
* If you don’t want to define the theme on every single execution add `THEME = "attila-master"` to the `pelicanconf.py`. 
* That is it. Now you can render the output on your local machine. 
* But that is not what we want!  
* We want to only write the markdown and save that in the Github repo. 
* The rest should be done by Netlify. 

## Publish Custom Theme to Netlify
* This of course does not work. The Attila theme is not available on the Netlify build machine.  
* How the hell does Netlify know what to do anyways? It must be analysing the repo, figure out that this is a python based project, install the requirements and then run the build command. That is nuts. 
* To make the Attila theme available on Netlify we need to make it available on the build machine. 
* One way would be to add the theme to the repo. Or configure the build environment.  

### Pulling Theme to Netlify Build Machine
* We could add this to the `Makefile`
```makefile
THEMEREPO?=https://github.com/arulrajnet/attila.git 
THEMECLONETARGET=/tmp/attila-$$RANDOM/attila/
GIT?=git

...

build:
	$(GIT) clone $(THEMEREPO) $(THEMECLONETARGET)
	$(PELICAN)-themes --install $(THEMECLONETARGET)
	$(PELICAN) $(INPUTDIR) -o $(OUTPUTDIR) -s $(CONFFILE) $(PELICANOPTS)
```
* Also, in the Netlify build settings define the build command to `make build`. 
* This pulls the latest version of the theme from GitHub, installs it on the Netlify build machine and generates the output.
* There are some downsides though to automatically cloning the latest master from GitHub. This means there could be a change to the theme that breaks your build. And you probably do not want to wait for the download of the latest theme version when you work on the local development.  

### Adding the Theme to the Blog Repo
* The possibly even easier option is to add the theme you want to have to the repo directly. 
* This does not even require the installation of the theme with `pelican-themes`.
* Right now, on the development machine, the Attila theme is installed. 
```shell
$ pelican-themes -l
simple
attila
notmyidea
```
* To remove it, run `pelican-themes -r attila`.
* Trying to build the site should run into an error because the theme is missing. 
```shell
$ make build
...
CRITICAL: Exception: Could not find the theme attila
make: *** [build] Error 1
```
* Create a `themes/attila` directory in the project
```shell
$ mkdir themes
$ mkdir themes/attila
```
* Copy the downloaded and unpacked theme folders content into the created directory.
```shell
$ cp -r ~/Downloads/attila-master/* themes/attila/
```
* Update the `pelicanconf.py` and change the `THEME` from `attila` to `themes/attila`.  You are now using the path rather than the installed name of the theme.
* Build the site `make html` (this command is automatically provided in the Makefile that is created during the `pelican-quickstart`).  
* You can remove the `build` command and the changes made to the makefile in the previous step. They are not needed anymore. 
* Make sure to update the build command in Netlify to `make html` to build the page the correct way. 

#dev/blog
#dev/courses/100DaysOfWeb in Python#