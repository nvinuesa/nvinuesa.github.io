---
layout: post
title: Upload multi subjects / session XNAT
date: 2015-09-05T15:18:01.000Z
categories: blog
---

Following the same line of work as the last blog post, I'll explain a possible way of uploading a set of Dicom images separated in folders indicating subject/session organization into [XNAT][xnat-site]. This solution is based on a script by Hauke Bartsch (hbartsch@ucsd.edu).

## Bash script:

{% highlight bash %}

# Create a session cookie, we want to re-use that session instead of providing

# login information every time. Number of sessions might be limited otherwise to 1000.

cookie=`curl -k -u $USER:$PASSWORD -X POST $XNAT/data/JSESSION` echo "Session ID is: $cookie"

# create subject in case it does not exist

echo "create subject $c" c=`curl --cookie JSESSIONID=$cookie -k -X PUT $XNAT/data/archive/projects/$PROJECT/subjects/$subject`

# create session in case it does not exist

echo "create session $c" c=`curl --cookie JSESSIONID=$cookie -k -X PUT $XNAT/data/archive/projects/$PROJECT/subjects/$subject/experiments/$session?xsiType=xnat:mrSessionData`

timestamps=( ) for file in `find $directory -type f -print` do

```
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
```

done echo "done sending files..." {% endhighlight %}

[jekyll]: http://jekyllrb.com
[jekyll-gh]: https://github.com/mojombo/jekyll
[xnat-rest]: https://wiki.xnat.org/display/XNAT16/Using+the+XNAT+REST+API
[xnat-site]: http://www.xnat.org/
