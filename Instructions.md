 1
Why the Amazon Dash Button is so hackable

Amazon's Dash Button is inexpensive -- and in order to keep costs down, the battery is nothing special. According to Matthew Petroff, it's good for about 1000 presses -- just long enough for the device to become obsolete, in theory.

In order to conserve battery life, the Dash Button is normally off. When you press the button, the Dash Button powers on and connects to the network.

This is where the magic lies: We can sniff network traffic and detect when your specific Dash Button attempts to connect to the network. Then, rather than ordering an item, we can perform some other command -- anything we want, really.

Not bad for $5.
2
Install or update Node.js onto your Raspberry Pi

Since we're going to use a Node module to emit events when the Dash Button is pressed, we first need to install Node.js.

Open Terminal (OSX) or Command Line (Windows) and connect to your Raspberry Pi using your Raspberry Pi's IP and the default username and password (assuming you haven't changed it):

ssh pi@your-pi's-ip

If you already have Node installed, give it an update. Otherwise, download the appropriate Node.js source:

Raspberry Pi 3:

wget https://nodejs.org/dist/v4.3.2/node-v4.3.2-linux-armv6l.tar.gz 
tar -xvf node-v4.3.2-linux-armv6l.tar.gz 
cd node-v4.3.2-linux-armv6l

Raspberry Pi 2:

wget https://nodejs.org/dist/v4.0.0/node-v4.0.0-linux-armv7l.tar.gz 
tar -xvf node-v4.0.0-linux-armv7l.tar.gz 
cd node-v4.0.0-linux-armv7l

Raspberry Pi A, B, B+, Zero:

wget https://nodejs.org/dist/v4.0.0/node-v4.0.0-linux-armv6l.tar.gz 
tar -xvf node-v4.0.0-linux-armv6l.tar.gz 
cd node-v4.0.0-linux-armv6l

Then, copy to /usr/local:

sudo cp -R * /usr/local/

Note:
You can check that you have the correct version of Node installed by running the command: node -v
Mentioned here:

    Find your Raspberry Pi's IP Address
    Raspberry Pi default username and password
    Change the password on your Raspberry Pi

3
Install node-dash-button

In order to detect when your Dash Button connects to the network so that we can run a program, we're going to use an awesome module called node-dash-button.

There are two packages to install -- libpcap for reading network traffic and node-dash-button for the fun stuff:

sudo apt-get install libpcap-dev
npm install node-dash-button

You may need to install npm manually before running the second command above:

sudo apt-get install npm

4
Set up your Dash Button

You'll need to go through the majority of the normal Dash Button setup.

To do this, download the Amazon app for iOS or Android and sign in using your Amazon account information.

Navigate to My Account > Set up a new device. Then, select Dash Button. Follow the on-screen instructions until you get to the "Select a product" step. Then, close out the app without selecting a product. Do not click the X to cancel setup -- just close the app.
Set up your Dash Button
Note:
If you've already paired your Amazon Dash Button with a product, no worries -- you can always remove the dash button through the same menu interface and start over. :)
5
Find your Amazon Dash Button's hardware address

We need to find the unique address for your Dash Button so that we can detect when it joins your network:

cd node_modules/node-dash-button
sudo node bin/findbutton

Your button's hardware address is separated:by:colons and should show up with a message of "Manufacturer: Amazon Technologies Inc." next to it. There may be multiple arp requests coming into your network (made more confusing if you have multiple Amazon devices in your home), so you might need to press the button a few times to identify it correctly.

Also, if you have issues detecting arp requests from your Dash Button, try plugging your Ethernet cable directly into your router.
6
Create a reboot (or shutdown) shell script

I'm going to use this Dash Button specifically for rebooting; however, if you want to use it to shut down your Pi instead, the instructions are the same -- you'll just need to use the appropriate Pi reboot or shutdown command in your reboot/shutdown shell script.

I'm going to create a reboot shell script to actually reboot the Pi; then, the node script will execute this shell script.

Why not run the reboot command directly instead?
I'm using this button to reboot a Pi that performs other commands (like turning off LEDs) prior to shutdown, so I like to contain my reboot commands in their own shell script so that I can execute some commands first.

After logging into your Pi, create your reboot or shutdown script in /usr/local/bin, which is a common directory for custom executables:

sudo nano /usr/local/bin/reboot.sh

Enter the following:

sudo reboot

Save and exit.
Mentioned here:

    Don't pull the plug: How to shut down or restart your Raspberry Pi properly

7
Create a node script to run the shell script

Now we'll create a node script that will use node-dash-button to execute the above shell script:

sudo nano node_modules/node-dash-button/dash-reboot.js

We can use this simple script:

var dash_button = require('node-dash-button'),
    dash = dash_button('xx:xx:xx:xx:xx:xx'), // replace xx:xx:xx:xx:xx:xx with your Dash Button's hardware address
    exec = require('child_process').exec;

dash.on('detected', function() {
    exec('sh /usr/local/bin/reboot.sh', function(error, stdout, stderr) {
        console.log('stdout: ' + stdout);
        console.log('stderr: ' + stderr);
        if (error !== null) {
            console.log('exec error: ' + error);
        }
    });
});

Save and exit.
8
Run the node script at system start

Lastly, we need to run the node script at system start so it will be watching for more reboot commands from your Dash Button after rebooting. Normally, an easy way to start scripts us to use crontab -- however, the node-dash-button script can fail with this method since the network hasn't fully initialized at the part of the boot process when the crontab runs.

Therefore, we're going to use a super simple process manager instead called Supervisor -- this manager will also restart the node script if it fails for some reason! Pretty handy.

Install Supervisor:

sudo apt-get install supervisor

Create the Supervisor config file:

sudo nano /etc/supervisor/conf.d/dash-reboot.conf

Add the following to the config file to tell it to run our node script:

[program:dash-reboot]
command=/usr/local/bin/node /home/pi/node_modules/node-dash-button/dash-reboot.js
directory=/home/pi/node_modules/node-dash-button/
autostart=true
autorestart=true
startretries=3
stderr_logfile=/var/log/supervisor/dash-reboot.err.log
stdout_logfile=/var/log/supervisor/dash-reboot.out.log
user=root

Save and exit.

Start Supervisor at system start:

sudo systemctl enable supervisor

9
Use your Amazon Dash Button to reboot your Pi!

Run our node script manually:

sudo node /home/pi/node_modules/node-dash-button/dash-reboot.js

Press your dash button. Your Pi should now reboot! After rebooting, press it one more time to ensure that your node script is now running at system start.

You're good to go!
