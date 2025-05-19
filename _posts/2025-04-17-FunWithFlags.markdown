---
layout: post
title:  "Fun With Flags"
date:   2025-05-17 16:00:00 +0000
categories: AI
---

I like neural networks. I think they are incredibly powerful and, while they have some obvious drawbacks like training data requirement, I think they have an incredible amount of Pros as well. This doc, however, is not an investigation into Neural networks. No, this is a ridiculous idea which came from a question - how small can a flag be before we can no longer identify what it is?

# Part 1 : How small can a flag be before we can no longer identify what it is?

This is quite a broad question - obviously the answer can differ greatly depending on what the flag is. Something like Japan doesn't need explaining - it's obvious that a red dot on a white background is going to be Japan, even when shrunk all the way down to 3x3 pixels. The same probably goes fro a country like Albania.

<img src="https://upload.wikimedia.org/wikipedia/commons/3/36/Flag_of_Albania.svg">

<img src="https://imgur.com/nEGqrBI.png">

This image here is obviously really small and distorted, but Albania has quite a recognisable flag and it's quite unique, so you can identify that this is albania.

Sticking to Europe, issues quickly arise with countries like Slovakia and Slovenia which have very similar flags - the difference be only in the crest on the left-hand side.

<img height="200px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/Flag_of_Slovakia.svg/1200px-Flag_of_Slovakia.svg.png"> 

<img height="200px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f0/Flag_of_Slovenia.svg/1200px-Flag_of_Slovenia.svg.png"> 

For a look at these high res SVG files, you can clearly see a difference, but when compressed down these will become harder and harder to distinguish.

My initial suggestion which I've essentially plucked from thin air is 6x10 pixels. From the initial samples I've attempted to draw, this seems to be sufficient enough to draw a decent enough flag that can be recognised by a human most of the time. 

# Part 2 : The AI

So how does this relate to AI? Well, if a human can identify my 6x10 flag drawings with ~50% accuracy, how well can an AI do? 

The goal is to use a neural network, training on a selection of flag drawings and see how well it can identify them. Optimally, we'll be able to identify these flags with greater than 50% accuracy. The main challanges here will be with similar looking flags like Slovakia and Slovenia, and the Netherlands and Luxemborg.

## 1. Binary Representation of the Flag

To get a flag ready for the AI, I have opted to use a 3-dimensional binary array. The x and y axis represent the coordinate on the pixel grid, and a third z-axis is used to represent teh colour of that pixel. In an array in the Z-axis, all items will equal - except the correct colour (where the 0th item means white, 1st means black, 3rd means red, etc..). I'll be limiting the amount of colours down to Black, White, Red, Green, Blue, Yellow, Orange, and a second lighter shade of blue. These list of colours should be sufficient to draw all the flags of europe (which is what we're starting with for now).

`<insert diagram here of 3d array representation>`

We can automatically create flag representations for all our testing and training data with a for loop and case statement.

{% highlight python %}
for pixel in flattened_flag:
    # flattened_flag here is a 1d representation of the flag
    z_axis = np.zeros(len(possible_colours)-1)
    match pixel.colour:
        case "black":
            z_axis[0] = 1
        case "white":
            z_axis[1] = 1
        case etc...
            
    binary_representation.add(z_axis)
{% endhighlight %}

The `flattened_flag` variable here is a 1-dimensional representation of the flag we're processing. Even though this does make the flag less human readable, it's fine to keep as long as we're consisten and the AI should be able ot process it fine.

<img height="200px" src="https://imgur.com/4S91Mjt.png">

## 2. Training


## 3. Testing

