---
title: Making Blog Using Jekyll Chirpy Theme on Windows OS
author:
  name: Dennis
categories: [Blogging, Tutorial]
tags: [Blogging]
pin: true
---

<h2>Introduction</h2>
As I started the github blog again, I would like to share the difficulties that I faced while setting up my blog.<br>
Among many [jekyll themes](https://jekyll-themes.com/free/), I like [Chirpy](https://jekyll-themes.com/chirpy/) theme the most and I chose this them this time. 

Other Jekyll themes could be used by fork to their own github repository and only by entering their blog url in _config.yml.
but the chirpy theme couldn't be set up that easily.<br><br>

<h2>Fork to Your Repository</h2>
Once, fork Chirpy theme source code to your github repository. 
![image_fork_1]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/fork_1.png)

And go to settings then change repository name to <span style="color:orange">[your github id].github.io</span>
![image_fork_2]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/fork_2.png){: width="300" height="300"){: .left}<br><br><br><br><br><br><br>


If you have moved the blog code to your repository as above, Now let's clone it to your local computer.
![image_clone]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/clone.png){: width="300" height="300"){: .left}<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

<h2>Edit _config.yml</h2>
Open _config.yml file and edit some settings
- title : Your Blog title.
- url:  Your blog url. <span style="color:orange">(the most important)</span> 
- avatar: your blog avatar image.   
<br><br>

<h2>Installation Ruby</h2>
I install [Windows OS version of Ruby](https://rubyinstaller.org/downloads/) because I have Windows OS.<br>
![image_installation_ruby]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/installation_ruby.png){: width="300" height="300"){: .left}<br><br><br><br><br><br><br><br><br><br>

After installation ruby, Enter number 3 in console window as below.<br>
![image_ruby_console]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/ruby_console.png){: width="300" height="300"){: .left}<br><br><br><br><br><br> 

Move to directory containing blog code and then installation gem dependencies.
![image_installation_bundle]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/installation_bundle.png){: width="300" height="300"){: .left}<br><br><br><br><br>
<br><br>

<h2>Installation WSL</h2>
After all gem dependencies are installed, You need to run init.sh script to initialize it.
But, Because shell script is only for Linux, You can't run init.sh directly on Windows. 
so you have to install WSL(Windows Subsystem for Linux) to run it on Windows.

Start -> Turn Windows features on or off -> Check Windows Subsystem for Linux
And restart Windows
And install Ubuntu in Windows Store and run it.
You need to create user. now you can run shell script.

<h2>Run init.sh</h2>
Go to the directory where your blog source code is stored.<br>
Enter the cmd command at windows file explorer address bar.<br>
Entering bash command in cmd, shell environment is created.<br>
Now, Run init.sh!<br>
If you want to know about init.sh, refer to the follow link [Chirpy Getting Started](https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Getting-started)

You need to set up user name and email for git because init.sh use git command when running. 
![image_set_git_info]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/set_git_info.png){: width="400" height="400"){: .left}<br><br><br><br><br><br>

If init.sh has run successfully, all you need to is push to your repository.
![image_init_successful]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/init_successful.png){: width="400" height="400"){: .left}<br><br><br><br>

Now, Go to action tab in your repository. you can identify what you have committed is building.
![image_building]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/building.png){: width="400" height="400"){: .left}<br><br><br><br><br><br>

It almost done!
After build is completed, Go to pages in settings.<br>
You need to change branch from master to gh-pages.<br>
gh-pages is a branch that is made just after it is built in action tab.
![image_change_branch]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/change_branch.png){: width="400" height="400"){: .left}<br><br><br><br><br><br><br>

It's done!!<br>
Let's go to your new blog. https://[your github id].github.io <br>
From now on, you can upload your post in blog.<br>
![image_new_blog]({{site.url}}/assets/img/2022-01-09-making-blog-using-jekyll-chirpy-theme-on-windows-os/new_blog.png){: width="300" height="300"){: .left}<br><br><br><br><br><br><br><br><br>