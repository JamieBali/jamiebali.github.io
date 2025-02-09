---
layout: post
title:  "GitHub Actions and Duolingo"
date:   2025-01-22 23:00:00 +0100
categories: github
---

Some GitHub profiles are really cool. Mine is trash. Let's fix that. Obviously we can add badges, and blocks made by other people, but I want something unique. Something a bit more stand out.

My plan was to create a display block showing my duolingo streak. Why? Becuase why not.

## Version 1 : A Web Host

The original plan I had for this was to create a webpage which takes a parameter in the URL for a Duolingo Username and a country name. We would then be able to throw together an image containing the country flag, the streak length, and Duo himself (That's the name of the Duolingo owl mascot).

This wasn't particularly difficult to produce and I made this as an SVG file.

# First, the duolingo API

{% highlight console %}
https://www.duolingo.com/2017-06-30/users?username=${username}&fields=streak,streakData%7BcurrentStreak
{% endhighlight %}

This here returns a json file containing info about a given user. We have a lot of options of what we can pull from the user, but we just care about the streak data. By digging into the json and pulling out `users[0].streak` we get a single for the current streak of the user.

We're building this as JS script so that it can be bundled into a webpage, so we can perform the API query using an `await fetch` async function, and then passing the result back to a constant.

# Second, the flag

In the future we can look at getting this going to work for other countries, but for now I'm just hard-coding the flag of the language I'm learning, Magyarul.

As we're building an SVG (and because the Hungarian flag is ultiamtely quite a simple tricolour), we can just create 3 rectangles within our SVG block

{% highlight html %}
<rect style="fill: rgb(206, 41, 57)" x="0" y="0" width="260" height="180" rx="38" ry="38"/>
<rect style="fill: rgb(71, 112, 70)" x="0" y="60" width="260" height="120" rx="38" ry="38"/>
<rect style="fill: white" x="0" y="60" width="260" height="60"/>
{% endhighlight %}

The other option here was to insert an image of the given flag - obviously this would work, but it would make the image less expandable - as an SVG we could scale this infinitely without the image becoming pixelated.

# Thirdly, putting the SVG together

To completely negate everything I've just said, I'm sticking a 1000x800 webp file of Duo into the SVN. This reduces the scalability of this image before it becomes pixelated. Creating an SVG of the duolingo owl is a little bit beyond my artistic ability, and I couldn't find a good svg file online, so this file works for now - it's also the exact pose I was envisioning. We can look at converting this to an SVG in a future version of this too.

With the flag, the streak number, and the image of duo all ready, we can string it all together into a single SVG. Flag on the base layer, image over the top, and the text on top of that. As before, building this with an SVG is easy enough to do and we can quite simply throw together a block which we append into the HTML block for the site.

# Lastly, assembling the site

This is where I encountered issues. To start with, I built a site as a basic image host - something so that you could stick in a url and it'd be able to pull out the image from that site as an embed (like imgur does). When copying this over to GitHub pages, the image host broke immediately.

In further testing, I came across another issue - when calling the image host from another site, the js that I attached here doesn't get run, meaning the image returns incorrectly anyway.

## Version 2 : Github Actions and SVG Files

Well with that disaster out of the way, let's do something else instead - produce the SVG in the local repo and have it display into github cover page as a direct image, rather than from an external site. Github profiles also can't execute Javascript so we'll need to produce the SVG in another way. The option I chose is GitHub Actions.

Github actions are a method of executing yml playbooks so that all manner of tasks can be executed. I use workbooks with RedHat's Ansible frequently where I work, so assembling this here wasn't too difficult.

{% highlight yml %}
jobs:
  generate-svg:
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3 

      - name: Run SVG generation script
        run: node generateSvg.js 

      - name: Commit SVG file
        run: |
          git config --local user.name "github-actions[bot]"
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git add generated.svg 
          git commit -m "Auto-generated SVG file"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
{% endhighlight %}

The schedule configured on this is just set up to run whenever a change is commit to the repo, but can also be executed manually. The next step will be getting this to run automatically every day.

# Finally, building the page

Now that we have an SVG file being commit into the repo by this workbook, we can just slap that local path into the github bio and it'll appear. The standard Github markup used for these pages allows us to insert basic HTML so we can just use a path like so:
 
{% highlight html %}
<img src=".\generated.svg"></img>
{% endhighlight %}

