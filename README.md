[Garage Door Controller](https://github.com/0-kaladin/garage-door-controller2)
======================

Monitor and control your garage doors from the web via a Raspberry Pi.

![Screenshot from the controller app][1] &nbsp; ![Screenshot from the controller app][2]

Overview:
---------

This project provides software and hardware installation instructions for monitoring and controlling your garage doors remotely (via the web or a smart phone). The software is designed to run on a [Raspberry Pi](www.raspberrypi.org), which is an inexpensive ARM-based computer, and supports:
* Monitoring of the state of the garage doors (indicating whether they are open, closed, opening, or closing)
* Remote control of the garage doors
* Timestamp of last state change for each door
* Logging of all garage door activity

Requirements:
-----

**Hardware**

* [Raspberry Pi](http://www.raspberrypi.org)
* Micro USB charger (1.5A preferable)
* [USB WiFi dongle](http://amzn.com/B003MTTJOY) (If connecting wirelessly)
* 8 GB SD Card
* Relay Module, 1 channel per garage door (I used [SainSmart](http://amzn.com/B0057OC6D8 ), but there are [other options](http://amzn.com/B00DIMGFHY) as well)
* [Magnetic Contact Switch](http://amzn.com/B006VK6YLC) (one per garage door)
* [Female-to-Female jumper wires](http://amzn.com/B007XPSVMY) (you'll need around 10, or you can just solder)
* 2-conductor electrical wire

**Software**

* [Raspbian](http://www.raspbian.org/)
* Python >2.7 (installed with Raspbian)
* Raspberry Pi GPIO Python libs (installed with Raspbian)
* Python Twisted web module

Hardware Setup:
------

*Step 1: Install the magnetic contact switches:*

The contact switches are the sensors that the raspberry pi will use to recognize whether the garage doors are open or shut.  You need to install one on each door so that the switch is *closed* when the garage doors are closed.  Attach the end without wire hookups to the door itself, and the other end (the one that wires get attached to) to the frame of the door in such a way that they are next to each other when the garage door is shut.  There can be some space between them, but they should be close and aligned properly, like this:

![Sample closed contact switch][3]

*Step 2: Install the relays:*

The relays are used to mimic a push button being pressed which will in turn cause your garage doors to open and shut.  Each relay channel is wired to the garage door opener identically to and in parallel with the existing push button wiring.  You'll want to consult your model's manual, or experiment with paper clips, but it should be wired somewhere around here:

![!\[Wiring the garage door opener\]][4]
    
*Step 3: Wiring it all together*

The following diagram illustrates how to wire up a two-door controller.  The program can accommodate fewer or additional garage doors (available GPIO pins permitting).

![enter image description here][5]

Software Installation:
-----

1. **Install [Raspbian](http://www.raspbian.org/) onto your Raspberry Pi**
    1. [Tutorial](http://www.raspberrypi.org/wp-content/uploads/2012/12/quick-start-guide-v1.1.pdf)
    2. [Another tutorial](http://www.andrewmunsell.com/blog/getting-started-raspberry-pi-install-raspbian)
    3.  [And a video](http://www.youtube.com/watch?v=aTQjuDfEGWc)!

2. **Configure your WiFi adapter** (if necessary).
    
    - [Follow this tutorial](http://www.frodebang.com/post/how-to-install-the-edimax-ew-7811un-wifi-adapter-on-the-raspberry-pi)
    - [or this one](http://www.youtube.com/watch?v=oGbDawnqbP4)

    *From here, you'll need to be logged into your RPi (e.g., via SSH).*

3. **Install the python twisted module** (used to stand up the web server):

    `sudo apt-get install python-twisted`
    
4. **Install the controller application**
        
    I just install it to ~/pi/garage-door-controller2.  You can install it anywhere you want but make sure to adapt these instructions accordingly. You can obtain the code via git by executing the following:
    
    `git clone https://github.com/0-kaladin/garage-door-controller2.git`

    To get any new changes to the code just `cd garage-door-controller2; git pull`
    
    That's it; you don't need to build anything.

5.  **Edit `config.json`**
   
    **options**
	- `auth` : Set to 'True' if you'd like to use the web authentication feature. Set to 'False' to disable web authentication.

    **smtp**

	Most of these are self explanatory.
	- `time_to_wait` : Value in seconds, after which a message will be sent stating the garage door was left open.

    **site**
	- `port` : The port the web application will be available on.
	- `username` : Username to use for web authentication.  (Ignored if auth is set to 'False') 
	- `password` : Password to use for web authentication.  (Ignored if auth is set to 'False')

    **doors**

	You'll need one configuration entry for each garage door.  The settings are fairly obvious, but are defined as follows:
	- `name` : name for the garage door as it will appear on the controller app.
	- `relay_pin` : The GPIO pin connecting the RPi to the relay for that door.
	- `use_state` : Set to False to ignore the state_pin if you have not wired up any tracking of door state.
	- `state_pin` : The GPIO pin conneting to the contact switch. (Be sure the use_state value above is True)
	- `approx_time_to_close` : How long the garage door typically takes to close.
	- `approx_time_to_open` : How long the garage door typically takes to open.

    The **approx_time_to_XXX** options are not particularly crucial.  They tell the program when to shift from the opening or closing state to the "open" or "closed" state.  You don't need to be out there with a stopwatch and you wont break anything if they are off.  In the worst case, you may end up with a slightly odd behavior when closing the garage door whereby it goes from "closing" to "open" (briefly) and then to "closed" when the sensor detects that the door is actually closed.

        
6.  **Control with init script**

    Configure the included init script.<br/>
    `sudo cp ~pi/garage-door-controller2/controller.sh /etc/init.d/controller`<br/>
    `sudo chmod +x /etc/init.d/controller`<br/>
    `sudo update-rc.d controller defaults`<br/>

    This will now enable you to stop/start/restart the controller as a normal program with
    `/etc/init.d/controller start` for example.<br/>
    It will also have it run at system startup and shutdown with the system.

    You can check if it is running with `/etc/init.d/controller status`
    
7. **Using the Controller Web Service**
The garage door controller application runs directly from the Raspberry Pi as a web service running on port 8080.  It can be used by directing a web browser (on a PC or mobile device) to http://[hostname-or-ip-address]:8080/.  If you want to connect to the raspberry pi from outside your home network, you will need to establish port forwarding in your cable modem.  
<br>
When the app is open in your web browser, it should display one entry for each garage door configured in your `config.json` file, along with the current status and timestamp from the time the status was last changed.  Click on any entry to open or close the door (each click will behave as if you pressed the garage button once).

TODO:
----------  
This section contains the features I would like to add to the application, but do not currently have time for.  If someone would like to contribute changes or patches, I would be all to happy to incorporate them.

* *Security*: Impose a configurable password on the web service.  Would need to discuss the best strategy (i.e., should we require the pw every time, or can the session persist on any given device which has authenticated).
* *New Feature*: Add a "close all" button to the bottom of the page to close all doors that have a state other than "closed" or "closing"
* *Configuration*: Make the port number a configuration option


  [1]: http://i.imgur.com/rDx9YIt.png
  [2]: http://i.imgur.com/bfjx9oy.png
  [3]: http://i.imgur.com/vPHx7kF.png
  [4]: http://i.imgur.com/AkNl6FI.jpg
  [5]: http://i.imgur.com/48bpyG0.png
  
