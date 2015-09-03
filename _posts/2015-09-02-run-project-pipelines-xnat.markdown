---
layout: post
title:  "Launch pipelines on an entire project in XNAT"
date:   2015-09-02 10:38:06 
categories: blog
---

[XNAT][xnat-site] (the eXtensible Neuroimaging Archive Toolkit) is a great tool for managing large sets of medical images (in particular neuroimaging). One of its useful features is the pipeline engine, which allows to easily write and launch pipelines, although it has a huge drawback: it is impossible to launch the same pipeline on all the subjects of a project. In this post I will explain one possible workaround using a very simple python script.

## XNAT is RESTful
If you are new to XNAT, you will discover that it manages imaging files in a custom fashion: PROJECT/SUBJECT/SESSION. Meaning that in your XNAT server you can have several projects (or imaging cohorts), with of course several subjects in it, and finally each subject with several imaging sessions (longitudinal studies) on it.
<br>On the other hand, the web application allows you to run pipelines from the UI; but as stated before, pipelines are only session-wide. This means that if you want to launch the same pipeline for every session of every subject in a project, you would have to manually do it by clicking the launch button on the UI.


+	<em>Easy to use and setup</em> - Jekyll has a huge range of documentation to get you started writing posts and the Simply Grey theme makes your blog look beautiful.
+	<em>Easy configuration</em> - I developed this theme in order to be as customisable as possible. If you want to add more links to the navigation bar, all you have to do is edit the _config.yaml file and the `urls` part of it.
+	<em>You can change it</em> - After being released with the MIT license (like Jekyll itself) you are free to change and basically do anything you want to this theme provided you keep the copyright notice in the files and distribute the license with it. 

## Jekyll
Jekyll is a static site generator developed in ruby that generates websites from markdown and many other formats. The benefit of this is that you can have a highly customisable blog where you can generate posts by writing easy markdown code whilst still retaining the small memory imprint that Jekyll has. 

### Code Snippets
Code Snippets are one of the main reasons why I love Jekyll and I think you will too. All code snippets become highlighted with great colours when you write the code in markdown. Here is an example of highlighted Ruby code in a weather application that I have made.
{% highlight bash %}
#!/usr/bin/env ruby

require 'json'
require 'net/http'
require 'libnotify'

def parsejson
    file = "http://api.openweathermap.org/data/2.5/find?q=London&mode=json"
    response = Net::HTTP.get_response(URI.parse(file))
    weatherjson = response.body
    actual = JSON.parse(weatherjson)

    # check for errors
    if actual.has_key? 'Error'
        raise "error with the url"
    end

    results = []

    actual["list"].each do |listitem|
        weather = listitem["weather"]
        weather.each do |weath|
            results.push(weath["description"])
        end
        main = listitem["main"]
        temp = main["temp"] - 273.15
        results.push ("%.2f" % temp)
    end

    return results
end

def notify(summary)
    Libnotify.show(:body => "Current temperature is: #{summary[1]} degrees celsius.\nCurrent description of conditions: #{summary[0]}", :summary => "Weather Update", :timeout => 10)
end

notify(parsejson())
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
[xnat-site]: http://www.xnat.org/
