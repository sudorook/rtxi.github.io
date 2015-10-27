---
title: Making New Modules in RTXI
categories: docs tutorials
layout: docpost
---

Outlined here is the development process for making new RTXI modules. We recommend our users to build their modules by using MyPluginGUI as the template. MyPluginGUI defines a basic GUI with customizable widgets and is found in the Plugin-Template directory of our GitHub repository. Anyone is free to fork off of it to create their own modules. GitHub provides [its own documentation](https://help.github.com/articles/fork-a-repo) on how to fork other users' repositories. We strongly recommend reading it. 

###The general process of modifying RTXI plugins is as follows:  

####1. Fork from RTXI's GitHub page  
From our site, go to the Plugin-Template repository. Contained there are the source files for MyPluginGUI, which is the default template from which we encourage users to develop.  

Forking will create a copy of our repository in your own GitHub account for you to modify. 

####2. Clone the repository on your local machine

Now, clone the repository. Developers for new modules are expected to base it on MyPluginGUI. Type:
{% highlight bash %}
$ git clone https://github.com/<username>/plugin-template plugin_template
{% endhighlight %}

Note that \<username\> represents the username of your own GitHub account, not that of RTXI. That is the case because users cannot modify repositories without the permission of their owners. Because MyPluginGUI was forked and copied into your own repository, you are completely free to edit it, and that will not affect the copy in the RTXI repository. 

In the command, the last parameter, `plugin_template`, sets the name of the directory in which all the cloned information will be stored. This parameter is optional. If left blank, the directory name defaults to the name of the repository being cloned. 

Now that clone is executed, look at the directory contents and you will see a subdirectory called `plugin_template`. Inside there are the source files for MyPluginGUI. An explanation of the header and source files is viewable [FIX THIS LINK!!!](https://github.com/RTXI/tutorials/wiki/MyPluginGUI-Base-Code). 

####3. Modify files
Modify the RTXI files to run the code you want. When you compile and install your module, be sure to rename the source files and classes to something other than `plugin-template` or `PluginTemplate`, respectively. It's preferable to name it  something that reflects what it does. A quick and easy way to change the names of the files, classes, and binary is to use the `sed` and `mv` commands:  
{% highlight bash %}
$ sed -i 's/plugin-template/<new_name_here>/g' plugin-template.* Makefile 
$ sed -i 's/plugin_template/<new_name_here>/g' plugin-template.* Makefile
$ sed -i 's/PluginTemplate/<NewNameHere>/g' plugin-template.* Makefile
$ mv plugin-template.cpp <new_name_here>.cpp
$ mv plugin-template.h <new_name_here>.h
{% endhighlight %}

The first `sed` command will search out any instance of the string `plugin-template` in your source files and replace them with a string you enter in place of the placeholder `<new_name_here>`. The next two `sed` commands will do the same. The `mv` commands will rename the files.  

Note that all commands that use the placeholder `<new_name_here>` assume that you provide the same string for each instance of the placeholder. Not following this convention *could* cause bugs that prevent your module from compiling if, for example, your sources file names and the `#includes` within them come out differently.  

Additionally, it's good practive to document your code. As a general rule, document it so that you're confident that anyone with programming experience will be able to understand what your code and overall module do.  

To install the module on your system, run:
{% highlight bash %}
$ make
$ sudo make install
{% endhighlight %}

####4. Push changes to your own GitHub repository

Commit your changes and push them back to your repository. (To learn what that means, [read this](/docs/tutorials/2015/04/07/how-to-use-git/).)  

This will install your module in RTXI, and it will be available among all of your installed modules when you use the 'Load Modules' in the drop-down menu for 'Modules' in the RTXI main window. 

####5. Submit a pull request for your plugin
Go to the home page for your repository and select 'Pull Requests' on the sidebar. This will take you to a page where you can submit pull requests to all of the projects to which you contribute and view the status of those requests. 

GitHub provides its own complete documentation on how to [create a pull request](https://help.github.com/articles/creating-a-pull-request) and [how to use them](https://help.github.com/articles/using-pull-requests) in general for project development. 