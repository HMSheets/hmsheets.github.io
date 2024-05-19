+++
title = "Nebula - Not Your Everyday VPN"
date = 2019-10-28

[taxonomies]
tags = ["Nebula", "VPN", "Networking"]
+++

# Nebula - Not Your Everyday VPN

[Nebula](https://github.com/slackhq/nebula) is a Mesh VPN that has recently been open sourced by Slack Technologies, the developers of the instant messaging service Slack.

This is interesting for a number of reasons, one of which is Nebula was developed  by Slack to connect their infrastructure accross regions without the overhead of a traditional VPN solution.

How is this relevant to us you might ask? Well instead of having devices connect directly to a centralized VPN server which handles all the connections, Nebula allows devices to connect directly to each other.

Wait! How does it do that?!?! Well that is the cool part. All devices connected to a Nebula network contact a server called a Lighthouse. A Lighthouse is like a DNS server in that the devices on the network ask a Lighthouse where another device is and then with that information the querying device can find the best, most direct path to the queried device it is trying to connect with. That means if another Nebula connected device is on the same local network the two devices will talk to each other locally on that internal network at internal network speeds. No need to route the traffic over the WAN for it to come right back into the network! 

With my limited testing I was able to connect my laptop, a couple of VMs on my hypervisor at home, and two web servers I have hosted on DigitalOcean. This allowed me to administer my home servers and hypervisor on the same flat `/24` network as my web servers in the cloud. That was mind blowing to me.

Nebula will allow me to set up seamless backups, logging, and monitoring on my more powerful hypervisor at home for all my servers regardless of where they are. This is not to say I did not run into any hiccups.

One hiccup I ran into is, like most VPNs, Nebula does not play well with a double NAT scenario. This is a problem for me because I live under someone else's roof and have my entire network behind my own firewall inside of someone else's network (you can find more info about double NAT [here](http://www.practicallynetworked.com/networking/fixing_double_nat.htm) if your curious). 

Nebula does technically penetrate the double NAT. A device can punch through both firewalls direct to the lighthouse but routing from one device directly outside the double NAT to a device buried deep inside the double NAT, by default, does not work well. I have not touched any of the Nebula Routing rules and I am still testing and tweaking my configs so it might still be possible to route traffic through the lighthouse to allow the two separated devices to communicate through double NAT.

Now that you have a primer on what Nebula is and why you might want to use it, you probably want to know how to set Nebula up. Below I will outline the steps I took to set up and test Nebula. This is not a comprehensive guide and I am pretty sure I didn't configure encryption while testing (which I intend to play with still).

## Installation 
  1. wget the latest release package for your platform from the [GitHub Releases](https://github.com/slackhq/nebula/releases) page 
      ```
      wget https://github.com/slackhq/nebula/releases/download/v1.1.0/nebula-linux-amd64.tar.gz
      ```
  3. Unpack the archive
      `tar -xf archive.tar.gz`
     1. There should only be two files that get unpacked, `nebula` and `nebula-cert`
  5. Move the nebula binary you unpacked to `/usr/local/bin/`
  6. If you do not have a CA created yet set one up with `./nebula-cert ca -name "Myorganization, Inc"`
  7. Now that you have a CA created we can sign individual host keys with `./nebula-cert sign -name "hostname" -ip "192.168.100.1/24" -groups "laptop,home,ssh"` and move it to `/etc/nebula`
     1. Note: If you want to allow a node to route untrusted routes, you need to include the subnets of the untrusted routs in the cert `./nebula-cert sign -name "hostname" -ip "192.168.100.1/24" -subnets "192.168.2.0/24" -groups "laptop,home,ssh"` (I am still testing this and have yet to get it to work)
  8. Create a new config.yml from the <a href="https://github.com/slackhq/nebula/tree/master/examples">example config</a> in the nebula git repo. Modify as needed and move to `/etc/nebula`
  9. Move `config.yml`, `ca.crt`, `{host}.crt`, and `{host}.key` to `/etc/nebula`
  10. Next copy the `nebula.service` from the above example files and modify as needed then add it to `/etc/systemd/system/`
  11.  Enable and start the systemd service

Repeat these steps for each node on the mesh. There is a flag in the config and corresponding settings that determine if a node is a lighthouse or not. The config has plenty of helpful comments in this regard. If you are still stuck and need some guidance, reviewing the readme on the GitHub page might point you in the right direction.

## Conclusion
And that my friends are the basics of installing and setting up Nebula. Like I mentioned before there are a ton of settings I have yet to test and I did not cover how to set up the encryption between nodes.

Overall I am very happy with Nebula so far and I look forward to learning some of its more advanced features.