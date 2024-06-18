# Project-Hand-Terminal
Wanted to document my "research" (fucking around and finding out) into what I think the future of consuemr tech will be, except self hosted. 

# But Why?
Because I wanted to.  Laptops as portable as they are, are not nearly as portable as smartphones, and not nearly as powerful as desktops can be.  So why can't we just use our smart phone to harnes the power of our Desktops while on the go?  That's bascially the whole question that started it all.  

I also love hacking and have always wanted to be able to poke at things at a moments notice where ever I may be.  So this project not only has to allow me to use the resources of my desktop, but also allow my desktop to talk to what ever network my phone is currently connected to as well.  These challanges were over come (and don't even require rooting your phone!!!) later on, we're focusing on the why right now.

As the descriptiong suggests another big reason is I wanted to get ahead of where consumer tech is going so I knew how to self host it when the time comes.  What do I mean by that though?  Great question voice in my head!

Basically I believe the future of consumer tech is we will all use "dummy terminals" with some capability of processing on device, but for the heavy stuff will offload it to some datacenter in "the cloud".  We already see that with some of Apple's new announcements at WWDC 2024, with siri having some impressive capabilities that are handled on chip, but when things are too hard will offload to Apple servers for Apple Intelligence, or to chat gpt.  I wanted that, but for myself and more manual because I'm a freak who loves to do things the hard way.


# High Level Overview
We will use a mesh VPN network in order to ensure our devices can talk at all times. Then SSH to access the computers/servers via fast CLI, Sunshine/Moonlight to access the desktop of our home computers, and microsocks to let us proxychains traffic from our computers at home through our phones. Termux will be the CLI environment we use on the phones to get it all set up and interact via SSH. At its core Android is still Linux. Termux lets us use it more like a "normal" linux distro by providing some userspace cli tools that are honestly pretty amazing. You can get alot running on Android these days including nmap, metasploit, dirb, and quite a bit more.  Android can actually be a pretty decent attack platform all on its own. It can be even better if you root it too! Unfortuneately I have a North American model Samsung Z fold 5, where Samsung has decided that we should not be able to unlock the bootloader... Europe models can... Asia models can... but nope not here in North America.... I'm not salty....


# Things you'll need
1. an android phone
2. a desktop, cloud vm, or home server to act as the heavy lifter
3. a [tailscale](https://tailscale.com/) account
4. the [F-Droid](https://f-droid.org/) app respository installed on your android phone
5. the termux app installed from fdroid (don't use the playstore one its no longer updated)
6. patience.

Go a head and sign up for tail scale and follow the instructions for adding your phone, and any other computer you want to access and use the resources with your phone for. The instructions on their site are pretty straight forward, and work well for even non-mainstream linux distros like Arch and EndeavorOS.

You'll also need to enable the SSH server on your desktop/server in order to remotely use it with your phone, for Systemd based Linux distros (most of them) it'll be a simple `sudo systemctl start sshd && sudo systemctl enable sshd` I'm not 100% sure how to do this on windows and mac but both come with openssh installed so it should be pretty easy to figure out using google.

Once you have all that set up the next part is to get the SOCKs5 server software for your phone. This is done with Termux using the command `pkg install microsocks`.  I highly recommend installing a few other tools, while we can proxy most of what we need from the desktop a few things work better without the proxy such as nmap for port scanning. You can install that with `pkg install nmap`. 

I also highly recommend customizing your termux environment, for example I installed fish becuase its my preferred shell, and to my surprise even using fish_config works like it does on normal linux, it opens your default web browser on your phone and lets you configure your fish shell!

Go a head and install Sunshine on your home PC, if you're doing this on a VM or in the Cloud you'll likely need to set up a headless xorg config to allow Sunshine to work.  I haven't done that myself so I'll leave that to you to Google.

All that's left to do now is tie it all together.

# Get it all together

Now that we have everything we need and everything is in the tailnet we can use it! 

First things first generate an ssh key on your phone in termux. While this isn't required it does make logging into your desktop at home easier. Open termux and run `ssh-keygen` after that we need to upload the public key to our desktop at home, luckily that's really easy too!  just run `ssh-copy-id user@desktophostname`. Now if you run `ssh user@desktophostname` in tail scale you should be logged into your your desktop over the tailnet, and you can do that anywhere in the world with an internet connection!

To exit the desktop CLI just type exit and you'll be back at your local termux session.  Now to enable SOCKs5. We already have MicroSocks installed, so we just need to run it! in Termux run `microsocks -p portnumer you want to listen on (must be above 1024)` for example if I wanted my proxy server running on port 31337 I'd run `microsocks -p 31337`. microsocks also supports authentication which is a good idea if you wanted to use it.  Now that microsocks is running we can ssh back to our desktop and install proxychains the normal way for your operating system. Once proxychains is installed edit the proxychains config file (loacated at /etc/proxychains.conf on most linux distros) at the very bottom delete the default proxy for TOR and add the config line for your phone, for example if my phone was at 1.1.1.1 on the tailnet (its not but for example sake) and listening on 31337 for the proxy server I'd ad the line `socks5 1.1.1.1 31337` if you use authentication add the username and password after that for example `socks5 1.1.1.1 31337 username password`

save and close the file and boom you can now proxy stuff using proxychains!  Just run `proxychains tool/command you want to proxy` and it'll do the rest!  If the output from proxychains is messing with you, you can quiet it using the -q flag `proxychains -q thing you want to run`.

If you need GUI access to your computer go a head and follow the setup instructions for sunshine on the desktop and moonlight on your phone.  The one thing I'll caution is that moonlight can not use the hostname of the desktop, you must copy the IP out of your tailnet console to set it up manually.  You can add a custom resolution to most linux distros either with a built in fuction of the DE/WM you're using, or if you're on Plasma on Wayland like I am then you can via kernel parameters. I'll add those when I boot up my desktop after work (my cat turned it off lmao).

# Next steps

Now that basic functionality is working and you can remotely access your desktop and run tools on it against the network your phone is connected to the sky is the limit.  I'll be continuing to work on cool tools/functionalities that should allow for a bit more API-like access to automate some of the things and make it easy.  For exmple below is a fish sript I wrote to start the microsocks server and then ssh to my desktop at home

```
function hackmode
  microsocks -p 31337 &
  ssh pyro@pyro-desktop
end
```

Then wrote this fish script to return to normal

```
function nomralmode
  killall microsocks
end
```

I'm planning on getting a gui set up to maybe automate some of this and make it a bit easier, but we'll see, for now this works well.

Huge shout out to tailscale for some of the really cool functionality, for example I have the mullvad addon that lets me set any mullvad server as an exit node, and can even use my phone as an exit node!  I'm still playing with that to see if I can use that instead of having to use SOCKs5 and proxychains, but haven't figured that out quite yet.
