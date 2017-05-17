Install or update Node.js onto your Raspberry Pi

Raspberry Pi 3:

wget https://nodejs.org/dist/v4.3.2/node-v4.3.2-linux-armv6l.tar.gz 
tar -xvf node-v4.3.2-linux-armv6l.tar.gz 
cd node-v4.3.2-linux-armv6l

Or

Raspberry Pi Zero:

wget https://nodejs.org/dist/v4.0.0/node-v4.0.0-linux-armv6l.tar.gz 
tar -xvf node-v4.0.0-linux-armv6l.tar.gz 
cd node-v4.0.0-linux-armv6l

Then, copy to /usr/local:

sudo cp -R * /usr/local/

Install node-dash-button
libpcap for reading network traffic and node-dash-button for the fun stuff:

sudo apt-get install libpcap-dev
npm install node-dash-button

You may need to install manually
sudo apt-get install npm

Set up your Dash Button

You'll need to go through the majority of the normal Dash Button setup.

To do this, download the Amazon app for iOS or Android and sign in using your Amazon account information.

Navigate to My Account > Set up a new device. Then, select Dash Button. Follow the on-screen instructions until you get to the "Select a product" step. Then, close out the app without selecting a product. Do not click the X to cancel setup -- just close the app.

Find your Amazon Dash Button's hardware address

We need to find the unique address for your Dash Button so that we can detect when it joins your network:

cd node_modules/node-dash-button
sudo node bin/findbutton



This repository contains the source code for the AIYProjects "Voice Kit". See
https://aiyprojects.withgoogle.com/voice/.

If you're using Rasbian instead of Google's provided image, read
[HACKING.md](HACKING.md) for information on getting started.

## Troubleshooting

The scripts in the `checkpoints` directory verify the Raspberry Pi's setup.
They can be run from the desktop shortcuts or from the terminal.
