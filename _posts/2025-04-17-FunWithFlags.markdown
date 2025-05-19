---
layout: post
title:  "Fun With Flags"
date:   2025-04-17 16:00:00 +0000
categories: AI
---

I like neural networks. I think they are incredibly powerful and, while they have some obvious drawbacks like training data requirement, I think they have an incredible amount of Pros as well. This doc, however, is not an investigation into Neural networks. No, this is a ridiculous idea which came from a question - how small can a flag be before we can no longer identify what it is?

<a href="https://colab.research.google.com/drive/1p_oYgl7_eXJ_t8qX6w4vQbHgE1Vte7M4?usp=sharing"> All code references is available here </a>

# Part 1 : How small can a flag be before we can no longer identify what it is?

This is quite a broad question - obviously the answer can differ greatly depending on what the flag is. Something like Japan doesn't need explaining - it's obvious that a red dot on a white background is going to be Japan, even when shrunk all the way down to 3x3 pixels. The same probably goes fro a country like Albania.

<img position="center" height="200px" src="https://upload.wikimedia.org/wikipedia/commons/3/36/Flag_of_Albania.svg">

While the flag of Albania has a relatively complex emblem, it is still very unique and is recognisable so I expect that it will be identifiable even at a very reduced size.

Even if we're just sticking to Europe, issues quickly arise with countries like Slovakia and Slovenia which have very similar flags - the difference be only in the crest on the left-hand side. Both of these flags are shown below.

<img position="center" height="200px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/Flag_of_Slovakia.svg/1200px-Flag_of_Slovakia.svg.png"> 

<img position="center" height="200px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/f0/Flag_of_Slovenia.svg/1200px-Flag_of_Slovenia.svg.png"> 

From a quick look large, high-res SVG files, you can clearly see a difference in teh two flags, but when compressed down these will become harder and harder to distinguish, to the point where a 3x3 flag will be practically impossible to distinguish at this size.

My initial suggestion (which I've essentially plucked from thin air) is flags which are 6x10 pixels. From the initial samples I've attempted to draw, this seems to be sufficient enough to draw a decent enough flag that can be recognised by a human most of the time. Very small details on the flags need to be recognised to correctly distinguish differences like Slovakia, Slovenia, and Croatia, but in general the other "complex" flag designs like North Macedonia, the UK, Bosnia and Herzegovina, and Cryprus which can't fit easily with enough detail into a 6x10 image are unique enough to be distinguishable.

# Part 2 : The AI

So how does this relate to AI? Well, if a human can identify my 6x10 flag drawings with ~50% accuracy, how well can an AI do? 

The goal is to use a neural network, training on a selection of flag drawings and see how well it can identify them. Optimally, we'll be able to identify these flags with greater than 50% accuracy. The main challanges here will be with similar looking flags like Slovakia and Slovenia, and the Netherlands and Luxemborg.

## 1. What Type of Network To Use.

My original plan here was to use a Convolutional Neural Network. These have been shown to excel in image classification tasks due to their ability to recognise edges and patterns (Kanellopoulos & Wilkinson, 1997). An example of how this would work is shown in the network diagram below.

<img position="center" height="200px" src="https://imgur.com/4S91Mjt.png">

The first convolution layer would probably have a convolution size of 3x3, then a dense layer, and finally a softmax layer. We'd then need to run this for each of the Red, Green, and Blue layers. Alternatively, we could produce a 3d representation of the flag and use a 3-d convolution to process the image.

I suspect that this convolution may not be optimal for identification of these flags. While the convlutions would definitely be able to classify Horizontal versus Vertical tricolours, it would probably struggle to then identify these flags based on their colours. I think that the colour problem is more important here, so instead I've opted to use substitute the first convloutional layer out for a second dense layer.

<img height="200px" src="https://imgur.com/4drEKs8.png">

The layer sizes in this diagram are inaccurate, but the structure here is correct. With the flags being 6x10, and having 8 possible colours, that gives us a starting size of 420. Our first Dense layer will go from a 420 size into another a 60 size (the 2-d size of the flag), and then into a 49 size (the number of unique flags in our Europe Dataset). From this final 49 size we'll be able to easily perform a softmax calculation to determine our predicted flag.

## 2. Binary Representation of the Flag

To get a flag ready for the AI, I have opted to use a 3-dimensional binary array. The x and y axis represent the coordinate on the pixel grid, and a third z-axis is used to represent teh colour of that pixel. In an array in the Z-axis, all items will equal - except the correct colour (where the 0th item means white, 1st means black, 3rd means red, etc..). I'll be limiting the amount of colours down to Black, White, Red, Green, Blue, Yellow, Orange, and a second lighter shade of blue. These list of colours should be sufficient to draw all the flags of europe (which is what we're starting with for now).

`<insert diagram here of 3d array representation>`

We can automatically create flag representations for all our testing and training data with a for loop and case statement.

{% highlight python %}
for pixel in flag:
    z_axis = np.zeros(len(possible_colours)-1)
    match pixel.colour:
        case "black":
            z_axis[0] = 1
        case "white":
            z_axis[1] = 1
        case etc...
            
    binary_representation.add(z_axis)
{% endhighlight %}

The `binary_representation` variable here is now a 1-dimensional representation of the flag. It is 420 items long (6x10x8) and can be passed directly into our network. 

## 3. Network Construction

I have used Keras to create my network as it is the system I have the most experience with and it provides a lot of quick and easy functions to produce recurrant neural networks. 

{% highlight python %}
  def __init__(self):
    self.model = keras.models.Sequential()
    self.model.add(Dense(420))
    self.model.add(Dense(60))
    self.model.add(Dense(49))
    self.model.add(Reshape((1,49)))
    self.model.add(Activation('softmax'))

    self.model.compile(loss='categorical_crossentropy', optimizer='adam')
{% endhighlight %}

I'll be using "categorical cross entropy" as my loss function. This function calcuilates loss based on the list the one-hot vector we provide. I am also using the "Adam" optimiser which uses a fairly simple gradient descent for optimisation.

## 4. Training
After convering all of our flag drawings (made by multiple different people so we have some amount of variety) into binary arrays we're ready to begin with the training. The training data here contains 4-5 examples for each flag for a total of 240 images. I've additionally drawn my own set of images - one for each flag - which I will be using as testing data.

{% highlight python %}
model.fit(np.array(tr_puzzles), np.array(tr_solutions), batch_size = 30, epochs=10, verbose=1)
{% endhighlight %}

Keras does most of the heavy lifting now, and we can very simply run this command to get the model training. The important parameters to pass in here are "batch size" and "epochs". For now I've chosen to use a batch size of 30. This means we'll be processing 30 images at a tim ebefore calculating teh loss and trying with new values. I've chosen to run for 10 epochs, meaning we'll be running through our full set of training data a total of 10 times.

{% highlight python %}
Epoch 1/10
8/8 [==============================] - 1s 5ms/step - loss: 3.7325
Epoch 2/10
8/8 [==============================] - 0s 5ms/step - loss: 1.7631
Epoch 3/10
8/8 [==============================] - 0s 4ms/step - loss: 0.8154
Epoch 4/10
8/8 [==============================] - 0s 5ms/step - loss: 0.4643
Epoch 5/10
8/8 [==============================] - 0s 4ms/step - loss: 0.3022
Epoch 6/10
8/8 [==============================] - 0s 3ms/step - loss: 0.2438
Epoch 7/10
8/8 [==============================] - 0s 4ms/step - loss: 0.1782
Epoch 8/10
8/8 [==============================] - 0s 4ms/step - loss: 0.1788
Epoch 9/10
8/8 [==============================] - 0s 4ms/step - loss: 0.1565
Epoch 10/10
8/8 [==============================] - 0s 4ms/step - loss: 0.1481
{% endhighlight %}

As shown in this output above, the loss in each epoch does reduce with every run through of the data. With this our model was fully trained in under a second, so we are able to begin our testing.

## 5. Testing and Optimisation

To begin with I ran through a test, getting the model to predict the flag for all of my testing examples. As expected we encounter some issues with Slovakia and Slovenia. What was less expected was issues between Switzerland and Denmark. I also made a complete oversight in that Austria and Latvia ended up identical due to the limitation on colour availability. We included a second share of blue to allow for Luxembourg, Azerbaijan, San Marino, and Kazhakstan to be drawn correctly, but didn't include an additional share of Red for Latvia.

The testing of this network resulted in a final accuracy of 42/49 or around 85%. I suspect that the main issue causing problems here is overfitting of data. As you can see in the training output, after epoch 7, the performance increases by much smaller amounts each epoch, so it's possible that after this point it is just overfitting to some of the example data. This explains why small differences, such as the shade of blue different people used in flags like Greece and Ukraine, cause these flags to fail to get recognised.

In order to optimise the performance of this network, I attempted running at some different epoch sizes. This graph below shows the calculated performance at these different epochs, peaking at a 46/49 or ~94% accuracy at 8 epochs.

`<insert graph here of performance vs epochs>`


