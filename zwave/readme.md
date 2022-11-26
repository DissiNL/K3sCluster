https://smarthome.community/topic/120/z-way-server-ubuntu-install

First start by making sure you have all the dependencies loaded and packages up to date.
```bash


sudo apt-get -y update && sudo apt-get upgrade
sudo apt-get -qy install libxml2 libarchive-dev curl
sudo apt-get -qy install sharutils tzdata gawk
sudo apt-get -qy install libavahi-compat-libdnssd-dev
sudo ln -s /usr/lib/x86_64-linux-gnu/libarchive.so.13 /usr/lib/x86_64-linux-gnu/libarchive.so.12
```
Then download and install z-way-server (current latest version is 3.0.6) and install it in the /opt folder
`
wget http://razberry.z-wave.me/z-way-server/z-way-server-Ubuntu-v3.0.6.tgz
sudo tar -zxf z-way-server-Ubuntu-v3.0.6.tgz -C /opt/
`
Then follow the auto start process here:
https://smarthome.community/topic/108/auto-start-z-way-server

For ubuntu 18 and later, there is currently an issue with z-way-server not supporting the newest libcurl4 causing apps download failures so the following additional steps are needed:

Force downgrade to libcurl3 and save a copy of it and then upgrade back to libcurl4

sudo apt-get -y install libcurl3
sudo cp /usr/lib/x86_64-linux-gnu/libcurl.so.4 /usr/lib/x86_64-linux-gnu/libcurl3.so.4.5.0
sudo apt-get -y install libcurl4 libcurl4-openssl-dev
stop the z-way-server if it is already running:

sudo systemctl stop z-way-server
edit the systemd service file

sudo nano /etc/systemd/system/z-way-server.service
And add the following text at the end of the Environment line after adding a space to separate from the previous quote:

'LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libcurl3.4.5.0'
Hit ctrl-o and ctrl-x to exit nano

and now start the server again after updating systemd:

sudo systemctl daemon-reload
sudo systemctl start z-way-server
