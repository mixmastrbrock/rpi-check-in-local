# Raspberry Pi as a Check-Ins Printer Station

This document outlines the steps I use to set up a Raspberry Pi as a Check-in Check-in Station. [Planning Center Check-Ins](https://planning.center/check-ins/download) Forked from seven1m's main repo created as a headless operation. Below is a revised version of his notes for this application:

This repo also includes a script that modifies the Pi and installs software.

## Why?

Raspberry Pi's are cheap, and helps reduce our hardware cost, compared to iPads, so deploy these on 400's with a small monitor or touchscreen attached.

## How it Works

We install the Planning Center Check-Ins app on the Raspberry Pi and set it to start automatically when the Pi boots up. We also install other necessary software and connect a local printer

## What You'll Need

1. A [Planning Center Check-Ins](https://www.planningcenter.com/check-ins) subscription

1. A Raspberry Pi 4 with at least 2 GB of RAM.

   This will *not* work on an older Raspberry Pi.

1. (optional) A Dymo printer

   I have only tested with the Dymo LabelWriter 450 and 450 Turbo.

   Other Dymo printers *should* work out of the box, but I haven't tested any others.

   Further: A printer made by another brand (not Dymo) could probably be made to work with this setup, assuming there is a Linux print driver for it. There is not any Dymo-specific software used, but the setup script only knows how to install Dymo printers. You would need to set up the printer manually or tweak the script.

## The Steps

1.  Enable Check-Ins "Universal Printing" [here](https://check-ins.planningcenteronline.com/universal_printing_beta).

1.  On a desktop computer, download Raspbian Buster from [here](http://downloads.raspberrypi.org/raspbian/images/raspbian-2020-02-14/). Note: this is the only version of Raspbian I have tested this with. If you choose a newer version, you're on your own!

1.  Use a tool like [Etcher](https://www.balena.io/etcher/) to write the downloaded image to your SD card.

1.  After the image is written to the SD card, open the "boot" partition. On Mac and Windows, the boot partition should appear as a drive in Finder/Explorer. You'll see files already there. If you don't see the boot partition, you may need to remove the SD card and re-insert it for the operating system to see the drive.

    Create a new file there called "ssh" or "ssh.txt". You can use a program like TextEdit or Notepad. The file does not need to have anything inside it. Just an empty file with the name "ssh.txt" or "ssh" is fine.

1.  Put the SD card in your Raspberry Pi 4, connect it to your physical network with an ethernet cable, then power it on.

1.  Determine the IP address of your Raspberry Pi. You can either do this from your router, if you can list DHCP leases there, or by using a tool like [nmap](https://nmap.org/).

    Here is how to search for your Raspberry Pi with nmap:

    ```sh
    sudo nmap -p 22 -open 192.168.1.0/24
    ```

    The last argument tells nmap about your local network. The most typical is `192.168.1.0/24`, but if you have another network IP address and range, adjust appropriately. For instance, if your network IP addresses are 10.0.x.y, then you might need to use `10.0.0.0/16`. You can use a tool like the [Subnet Calculator](http://www.subnet-calculator.com/) to help.

    The resulting output will look something like this:

    ```
    Nmap scan report for 192.168.11.9
    Host is up (0.89s latency).

    PORT   STATE SERVICE
    22/tcp open  ssh
    MAC Address: 80:2A:A8:68:24:53 (Ubiquiti Networks)

    Nmap scan report for 192.168.11.10
    Host is up (1.0s latency).

    PORT   STATE SERVICE
    22/tcp open  ssh
    MAC Address: 80:2A:A8:7C:05:4D (Ubiquiti Networks)

    Nmap scan report for 192.168.11.179
    Host is up (0.065s latency).

    PORT   STATE SERVICE
    22/tcp open  ssh
    MAC Address: 00:11:24:21:19:FA (Apple)

    Nmap scan report for 192.168.11.207
    Host is up (0.0090s latency).

    PORT   STATE SERVICE
    22/tcp open  ssh
    MAC Address: B8:27:EB:EE:C9:A3 (Raspberry Pi Foundation)

    Nmap done: 256 IP addresses (30 hosts up) scanned in 15.45 seconds
    ```

    You can see from the output, that only one device on my network responds to the SSH port 22 **and** has a Mac Address assigned to Raspberry Pi Foundation. This is the Pi I'm looking for!

1.  SSH to the Pi and change the default password. Be sure to substitute the IP address with the one you found in the step above.

    ```sh
    ssh pi@192.168.X.Y
    ```

    Enter the default password: `raspberry`

    ```sh
    passwd
    ```

    Enter `rapsberry` one last time, then enter your new desired password.

1.  Download and run the setup script.

    Note: You should still be connected to the Pi. This command will run remotely on the Pi itself.

    ```sh
    curl https://raw.githubusercontent.com/mixmastrbrock/rpi-check-in-local/master/setup.sh -o setup.sh
    chmod +x setup.sh
    ./setup.sh
    ```

1.  Assuming you made it through the setup process and you were able to print a test label, you can continue to the next step.

    **DO NOT CONTINUE until you can get a test label to print using the setup script above.**

1.  Set up the Planning Center Check-Ins Software.

    You should see Planning Center Check-Ins asking you to set up a new station.

    Go through the process to create a new station. Give it a name, and accept the defaults for everything else.

    *For the station name, I use something like "Welcome Center RPi" so I know where it is and what type of station it is.*

1.  You should see this dialog window:

    <img src="images/qz_prompt.png" alt="qz prompt" height="180">

    Be sure to select "Remember this decision" and click the "Allow" button.

1.  Do a test check-in!

## Troubleshooting

**DO NOT CONTACT PLANNING CENTER SUPPORT ABOUT ISSUES WITH THIS SETUP. THEY WILL NOT BE ABLE TO HELP YOU.**

**Problem: Printing doesn't work!**

Here are some troubleshooting steps, starting closest to the hardware:

1.  Make sure the setup script was able to print a test label. If not, you may need to figure out why the command `echo "test" | lpr -P "Dymo" -` isn't working.

1.  Check that the Dymo printer is setup via Cups at [https://IPADDRESS:631](https://IPADDRESS:631). Print a test page from this UI.

1.  Use VNC to connect to the Pi and visit the Print Setup page in the Check-Ins app. You can get there by pressing Ctrl-2.

**Problem: Labels print slow and/or cut-off**

Sometimes, labels will print **extremely** slow and pause mid-way through printing a label.

I have found that _some_ Dymo printers do not work well with my `dymo_lag_fix.rb` script. It's not an issue with a particular model, but rather seems to be a random problem. My solution is to swap printers around (often times even the same exact model number) and everything starts working as expected!
