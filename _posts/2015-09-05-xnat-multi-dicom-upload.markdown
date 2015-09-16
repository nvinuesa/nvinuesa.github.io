---
layout: post
title:  "Upload multi subjects/session XNAT"
date:   2015-09-05 15:18:01 
categories: blog
---



{% highlight bash %}
    # Create a session cookie, we want to re-use that session instead of providing
    # login information every time. Number of sessions might be limited otherwise to 1000.
    cookie=`curl -k -u $USER:$PASSWORD -X POST $XNAT/data/JSESSION`
    echo "Session ID is: $cookie"
    
    # create subject in case it does not exist
    echo "create subject $c"
    c=`curl --cookie JSESSIONID=$cookie -k -X PUT $XNAT/data/archive/projects/$PROJECT/subjects/$subject`
    # create session in case it does not exist
    echo "create session $c"
    c=`curl --cookie JSESSIONID=$cookie -k -X PUT $XNAT/data/archive/projects/$PROJECT/subjects/$subject/experiments/$session?xsiType=xnat:mrSessionData`
    
    timestamps=( )
    for file in `find $directory -type f -print`
    do 
    
         # move file over using REST API
         c=`curl  --cookie JSESSIONID=$cookie -s -k -H 'Content-Type: application/dicom' -X POST "$XNAT/data/services/import?inbody=true&PROJECT_ID=$PROJECT&SUBJECT_ID=$subject&EXPT_LABEL=$session&prearchive=true&overwrite=append&format=DICOM&content=T1_RAW" --data-binary @$file | tr -d [:cntrl:]`
         echo -n "."
         timestamp=`echo $c | cut -d'/' -f6`
         # is timestamp new?
         found="0"
         for f in "${timestamps[@]}"; do
            if [ "$f" = "$timestamp" ]; then
    
               found="1"
            fi
         done
         # add to array
         if [ $found = "0" ]; then
            timestamps+=($timestamp)
            echo "found a new series $timestamp"
         fi
    
    done
    echo "done sending files..."
{% endhighlight %}
## Conclusions
With this very simple python script we are able to overcome a "limitation" in XNAT for some of us and at the same time we can experiment with the REST API.
<br> The entire script can be found here: 

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
[xnat-site]: http://www.xnat.org/
[xnat-rest]: https://wiki.xnat.org/display/XNAT16/Using+the+XNAT+REST+API
