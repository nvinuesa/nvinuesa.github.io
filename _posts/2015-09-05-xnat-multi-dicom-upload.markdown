---
layout: post
title:  "Upload multi subjects/session XNAT"
date:   2015-09-05 15:18:01 
categories: blog
---

[XNAT][xnat-site] (the eXtensible Neuroimaging Archive Toolkit) is a great tool for managing large sets of medical images (in particular neuroimaging). One of its useful features is the pipeline engine, which allows to easily write and launch pipelines, although it has a huge drawback: it is impossible to launch the same pipeline on all the subjects of a project. In this post I will explain one possible workaround using a very simple python script.

## XNAT is RESTful
If you are new to XNAT, you will discover that it manages imaging files in a custom fashion: PROJECT/SUBJECT/SESSION. Meaning that in your XNAT server you can have several projects (or imaging cohorts), with of course several subjects in it, and finally each subject with several imaging sessions (longitudinal studies) on it.
<br>On the other hand, the web application allows you to run pipelines from the UI; but as stated before, pipelines are only session-wide. This means that if you want to launch the same pipeline for every session of every subject in a project, you would have to manually do it by clicking the launch button on the UI.

Hopefully, XNAT's REST API is quite complete (some documentation can be found [here][xnat-rest]). This means that we can for instance retrieve the list of projects in the site, the list of subjects in one project, the list of pipelines, etc. 
<br> The first step in the script is therefore to get the list of projects from XNAT using the REST API:
{% highlight python %}
# Obtain a list of projects present at the site:
request = urllib2.Request(site + "/REST/projects?format=json")
base64string = base64.encodestring('%s:%s' % (user, password)).replace('\n', '')
request.add_header("Authorization", "Basic %s" % base64string)   
response = urllib2.urlopen(request)
html = response.read()
data = json.loads(html)
projectsIt = data['ResultSet']['Result']
{% endhighlight %}
After the retrieved data is converted back to json, is easy to pick up one desired element on the resulting dictionary. Following this procedure, we can therefore get the list of pipelines that are available for the chose project (because in XNAT you have to make a pipeline "available" in each project you want to use it on) in the same fashion:
{% highlight python %}
# Obtain a list of pipelines present at the chosen project:
request = urllib2.Request(site + "/data/projects/" + project + "/pipelines?format=json")
base64string = base64.encodestring('%s:%s' % (user, password)).replace('\n', '')
request.add_header("Authorization", "Basic %s" % base64string)   
response = urllib2.urlopen(request)
html = response.read()
data = json.loads(html)
pipeIt = data['ResultSet']['Result']
{% endhighlight %}
Again, in the same way, we choose a pipeline and the last step before launching it is to get the input arguments needed for that particular pipeline (since the user will have to provide this arguments and he/she may not know which are these input arguments):
{% highlight python %}
# Obtain the list of input parameters on the chosen pipeline:
request = urllib2.Request(site + "/data/projects/" + project + "/pipelines/" + pipeline)
base64string = base64.encodestring('%s:%s' % (user, password)).replace('\n', '')
request.add_header("Authorization", "Basic %s" % base64string)   
response = urllib2.urlopen(request)
html = response.read()
data = json.loads(html)
argIt = data['inputParameters']
{% endhighlight %}
## Launch in T-minus 10,9,...
After collecting all the previous data from the user, it is time to launch the pipeline on the entire project. Of course, this is also done via XNAT's REST API. 
<br> There is still one last detail; since the goal of this script is to launch a pipeline on ALL the subjects of a particular project, the user should not even have to know how many or which are the subjects in the chosen project. Therefore we will get the list of subjects in a project from xnat withouth letting the user select any subgroup of it.
<br> Here's the function:
{% highlight python %}
def launch(site, user, password, project, pipeline, args):
    request = urllib2.Request(site + "/data/archive/projects/" + project + "/experiments?format=json")
    base64string = base64.encodestring('%s:%s' % (user, password)).replace('\n', '')
    request.add_header("Authorization", "Basic %s" % base64string)   
    response = urllib2.urlopen(request)
    html = response.read()
    data = json.loads(html)
    experiments = data['ResultSet']['Result']
    for exp in experiments:
        print('Launching ' + pipeline + ' on experiment ' +exp['ID'])
        request = urllib2.Request(site + "/REST/projects/" + project + "/pipelines/" + pipeline + "/experiments/" + exp['ID'] + "?" + args)
        request.add_header("Authorization", "Basic %s" % base64string) 
        response = urllib2.urlopen(request, '')
        html = response.read()
{% endhighlight %}
## Conclusions
With this very simple python script we are able to overcome a "limitation" in XNAT for some of us and at the same time we can experiment with the REST API.
<br> The entire script can be found here: 

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
[xnat-site]: http://www.xnat.org/
[xnat-rest]: https://wiki.xnat.org/display/XNAT16/Using+the+XNAT+REST+API
