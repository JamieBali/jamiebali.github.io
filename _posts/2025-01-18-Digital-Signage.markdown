---
layout: post
title:  "Digital Signage Driven Through Raspberry Pis"
date:   2025-01-18 13:00:00 +0100
categories: raspberry-pi
---

A few months ago I was investigating some digital signage solutions for work and came across a lot of resources and options that didn't fully fit the restrictive conditions I needed to produce a solution in. That aside, something like this would be helpful on *the commune* so I built an in-house solution.

## Part 1 : Hardware

Obviously we're using an army Raspberry Pi microcomputers to do this. Who do you think I am?

The plan was to set up 2 different TVs. The screens we picked up were some old samsung TVs. They aren't smart or anything, so can't use the smart things IoT app or anything, but that's good because (and I may have mentioned this before) Raspberry Pi. 

These TVs are good because they will a) support Raspberry Pi input through HDMI, b) can we set up to support WOL which is a feature we want for automation and energy efficiency, and c) have a USB port that can power a Pi.

The Pis we're using are Raspberry Pi Model 3 B+. We've picked to use these because I had one spare that hadn't been used in my cluster, and I was able to pick up a 2nd on ebay for £18. The 3 B+ is a high enough spec to run a Linux distro with a GUI that can display 1080p images with ease, and will be able to fetch remote images easily. 

The Pi 3 B+ can apparently be boosted to support 4k output, but we'll have to see how that goes later.

## Part 2 : Pi Setup

First things first, configuring the Pi. These machines are running standard 64-bit Raspbian with full GUI. After going through a very basic install using the RPi imager for Windows, I had the two devices up and running.

Before actually making any software, we need to make sure that these are 1) Set up with a static IP, 2) have python installed and updated, 3) have reboots configued (as they'll be 'always-on' machines), and 4) are mounted with both ethernet and power.

# 1. Setting up a static IP

Setting a static IP on the Pi is done through DHCP. If you have a private DHCP server it can be done there, but for us we have a lovely switch on our router saying "use this router as a DHCP server", so that's nice and easy.

1\. Load the machine locally, with a mouse and keyboard and a monitor, so you can open up a console.

2\. run `ifconfig` to get all your interface info.

3\. find the linesstarting `inet` and `ether` for the interface you want.

{% highlight console %}
[Interface]:
    inet xx.xx.xx.xx  netmask 255.255.255.0  broadcast 192.168.0.255
    ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
{% endhighlight %}

These numbers are your `ip address` and `mac address` respectively, and can be used to set up your machine in DHCP.

4\. log into your DHCP server and set up your IP reservations. I'm happy to stick with the IP I was assigned by default.

Once these steps are done, you'll need to reboot your machine to make sure that static IP is set.

# 2. Have Python installed and updated

Now that an IP is set we can put away this yucky UI and get out the beatiful terminal.

0\. Bonus step I'm adding in after the fact - it may be helpful to run `sudo apt update` before the install of python to make sure you've got all the required modules to actually build python

`sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev  libsqlite3-dev`

1\. Run `python --version` to see what version we have installed. We've got 3.11.2 which should be plenty for what we want to run, but we're going to upgrade to 3.12 to be safe.

2\. Download the most up-to-date version of python by running `wget https://www.python.org/ftp/python/3.12.4/Python-3.12.4.tgz` 

3\. This will download the tar file, so we need to extract this using `tar -xf Python-3.12.4.tgz`

4\. Now we've got this extracted we can delete the tar file and cd into the directory

{% highlight console %}
Jamie@SensiblyNamedPi-1:~ $ rm Python-3.12.4.tgz
Jamie@SensiblyNamedPi-1:~ $ cd Python-3.12.4/
Jamie@SensiblyNamedPi-1:~/Python-3.12.4 $
{% endhighlight %}

5\. run `./configure --enable-optimizations` and wait for the installer to check over the system and prepare the file system

6\. run `sudo make  && sudo make altinstall` to install the python version onto the Pi

7\. this will have installed python with the execute call being `python3.12`. You can test this by running `python --version` This is fine, and we can just set our lines to run like that if we want, or we can set it to run 3.12 when we just call `python` normally. 

The remaining steps here are optional:

8\. remove any existing softlink to the python bin by running `sudo rm /usr/local/bin/python` 

9\. replace the softlink to your new version by running `sudo ln -s /usr/local/bin/python3.12 /usr/local/bin/python`

# 3. Reboots

Oh reboots, my best friend. The saviour of all. Fixer of 99% of the issues I've had with any machine. Turns out machines that aren't designed to be always-on should't be always-on, and the Pi is no exception.

I've set up reboots on linux machines before so it's as simple as running `sudo nano /etc/crontab` and adding a new line for the reboot

I like to do reboots at 4am as that is definitely the deadest time. We have people up until 3am sometimes, and people awake at 5am sometimes, so 4am is a nice middle ground. Not that these digital signage machines really care about the time they reboot as long as it's not mid-day

I have added a new crontab line very simply

`0  4    * * *   root    reboot`

# 4. Mounting, Ethernet, and Power

We've fed an ethernet cable from the nearest port to the TV, hiding it in the cornice so it looks nice.

Power to the TV comes from a cable attached to the wall under a plastic sheath, and the power to the pi comes directly from the TV.

To mounth the Pi to the TV, I've 3d-printed a small basket which hooks onto the mounth we've used to attach the TV to the wall.

## Part 3 : The Code

So how is this going to work?

Firstly, I've created a flowchart.

<img src="https://imgur.com/A4NCvtZ.png"></img>

This shows a very basic overview of the process, meaning we need 3 functions: one to get the image from the fileshare, one to display the image, and one to sleep for a short period of time.

I've stuck the code for this here: <a href="https://github.com/JamieBali/DigitalSignage">Digital Signage Repo on Github</a>

# Pulling images

We use the `os` and `PIL` libraries to grab the images from a remote fileshare to display. Currently the images display in a random order as the `get_image` function is set to do so. Alternatively, we could get the system to pull these iamges sequentially if we wish.

{% highlight python %}
    # Randomly select an image path
    image_path = random.choice(os.listdir("\\\\[Storage Machine Path]\\NVMe\\DigitalSignage\\"))
{% endhighlight %}

These images are re-scaled if required so that they fit to the *height* of the screen. We're maintaining the image ratio to ensure that there isn't stretching which will cause text to be unreadable - the suggestion is that all monitors should be the same dimensions so that a template can be provided for all displayed images.

{% highlight python %}
    # Load the image using PIL
    # We have to append the storage machine path again as the random.choice only returns the file name.
    image = Image.open("\\\\[Storage Machine Path]\\NVMe\\DigitalSignage\\" + image_path)
    iw = image.width
    ih = image.height
    h = root.winfo_screenheight()
    ratio = iw / ih                 # Calculate ratio to ensure image doesn't get stretched
    w = int(ratio * h)
    image = image.resize((w, h))    # Resize the image if needed
    photo = ImageTk.PhotoImage(image)
{% endhighlight %}

# Displaying images

Images are displayed on a UI created through the `tkinter` library. The UI created here is very quick and responsive, and by loading the images in the correct way we can seamlessly transition between two images. As seen in the code block below, we have to load in the image we want to display next *before* we destroy the previous image, then we immediately go to display it.

Placing image destruction before displaying the image resulted in a fraction of a second of delay as the image is downloaded, loaded, and resized before being displayed which resulted in a jarring flash.

{% highlight python %}
    # Create a label to display the image
    image_label = tk.Label(root, image=photo)
    image_label.pack()        

    # Display the image for a given number of seconds
    root.update()
    time.sleep(120) # this is a time in seconds
    
    # call function to load new image
    photo = get_image()

    # Remove the image
    image_label.destroy()

    # Schedule the display_image immediately, as the last image just got destroyed.
    root.after(0, display_image(photo))
{% endhighlight %}

## Part 4: Installing this on the machine

Firstly, there are a few packages we need to get installed

{% highlight console %}
    sudo apt-get install python3-tk
    sudo apt-get install python3-pil
{% endhighlight %}

Next, deploy out the code however you want. I was able to git pull it, but you could WinSCP or ftp it if you want.
You will need to edit the code to make sure we're pointing to the right display paths. Additionally, we have the option to adjust any of the other script parameters like display time, or add a delay in the display of images (so time where no image is displayed if you want that).

Lastly the script needs to be scheduled to execute on startup.

{% highlight console %}
    sudo nano /etc/rc.local
{% endhighlight %}

Then, add the line you want to execute on startup from here. Currently we have machines which use rc.local to start up the NAS with mounted drives, or the media server to re-connect to any network services and accounts.

Since this script executes continuously, you need to make sure to *fork the process* by adding an Ampersand (&) to the end of the line in rc.local

{% highlight python %}
    sudo python /home/pi/main.py &
{% endhighlight %}

## Further Development

In the worry that this is becoming too long, I've had the following ideas and suggestions provided since I set this up.

1\. Can we display videos?
Currently, no we can't. I'm not entirely sure if tkinter can support videos or gifs, and I'm fairly certain PIL can't. It's worth investigating if this is something we'd actually want to be doing, but not 100% sure if we actually have any videos we'd want to display here.

2\. Images sometimes stay up for longer than 2 minutes
This isn't true - what is actually happening is the random image selection is picking the same image twice in a row, so maybe we need to implement something to prevent the same image from repeating like this.

3\. Is there a better way we can manage this instead of just putting file into a fileshare?
Yes, probably, and I have some *ideas* which you'll see in version 1.1, coming at some point maybe.

4\. Installation is a pain. Can we set up switches or clps?
Yes, that's a very good idea. Having to edit the code manually makes it hard to use, so we should have CLPs for DisplayTime, ImagePath, and DisplayMode (random or sequential)
