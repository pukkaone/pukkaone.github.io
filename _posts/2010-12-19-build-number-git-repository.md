---
layout: post
title: Build Number From Git Repository
---

A build number is an identifying number assigned to a software release.  The
software displays the build number to the user in some fashion, such as in an
About dialog.  Subsequent releases should have increasing build numbers (but
the build numbers don't have to be contiguous), so you can say to a customer,
"If you have build number *N* or higher, then you have the fix to that bug."

If your source code is in [Subversion](http://subversion.apache.org/),
then an obvious choice for the build number is the repository revision number
checked out to build the release.  Getting a build number
from [Git](http://git-scm.com/) is not so obvious.  I'm going to outline a
scheme using the
[git describe](http://www.kernel.org/pub/software/scm/git/docs/git-describe.html)
command, which counts how many commits are tranversed to reach a tag.  Create a
tag pointing to some commit in the history.  A likely candidate is the first
commit in the repository.  The build number is the number of commits between
the current commit and the tag.  To guarantee an increasing build number, all
these conditions must be satisfied:

 1. Make all releases from the same Git repository.
 1. Make all releases from the same branch (master in my case).
 1. Do not rewrite history. 

Create a tag named `build` pointing to the first commit in the repository:

    % git tag -a -m "For calculating build number" build `git rev-list HEAD | tail -1`

The `git describe` command by default searches to the nearest tag.  Use the
`--match` option to specify the tag name.  Trying the command, you see your
build script needs to extract the build number from the output (`12` in the
example):

    % git describe --match build
    build-12-g53e5502

This [Ant](http://ant.apache.org/) task extracts the build number into the
`BUILD_NUMBER` property:

{% highlight xml %}
<exec executable="git" outputproperty="BUILD_NUMBER">
  <arg value="describe"/>
  <arg value="--match"/>
  <arg value="build"/>
  <redirector>
    <outputfilterchain>
      <tokenfilter>
        <replaceregex pattern="^[^-]+-" replace=""/>
        <replaceregex pattern="-.+$" replace=""/>
      </tokenfilter>
   </outputfilterchain>
 </redirector>
</exec>
{% endhighlight %}

This [CMake](http://cmake.org/) script parses the build number into the
`BUILD_NUMBER` variable:

{% highlight cmake %}
find_package(Git)
if(GIT_FOUND)
    execute_process(
            COMMAND ${GIT_EXECUTABLE} describe --match build
            OUTPUT_VARIABLE DESCRIBE_BUILD
            OUTPUT_STRIP_TRAILING_WHITESPACE)
    string(REGEX MATCH "[0-9]+" BUILD_NUMBER ${DESCRIBE_BUILD})
endif()
{% endhighlight %}
