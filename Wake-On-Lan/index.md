# Wake-On-Lan (WOL)
- [What it is](#what-it-is)
- [Why would I want it enabled](#why-would-i-want-it-enabled)
- [How it works](#how-it-works)
- [Project](#project)
- [Sources](#sources)

## What it is

Wake-On-Lan (WOL) is a feature that allows a computer to be turned on (woken up) from a remote location using a network message.
It works on shut down and sleep mode.

## Why would I want it enabled

### Pros
- **Remote Access**: You can access your files or remote desktop into your computer without leaving it on 24/7.
- **Energy Saving**: Keep your computer sleeping or off when not in use, and wake it up only when needed.
- **Maintenance**: IT admins can wake up computers at night to push updates or backups.

### Cons
- **Security Risk**: If someone gains access to your network, they could wake up your devices and access them.
- **Complexity**: Requires BIOS/driver configuration and sometimes port forwarding for WAN access.
- **Power**: Even when off, the network card draws a small amount of power to listen for the magic packet.

## How it works

The computer has a special network interface card (NIC) that can receive a magic packet.
The magic packet is value is always `'ff'` repeated 6 times, followed by the MAC address of the computer repeated 16 times.
It's sent using the UDP protocol to port 9 as a broadcast message.
Only the device with the same MAC address will wake up.
Usually devices have their WOL feature disabled by default, on most BIOSes you can enable it in the BIOS settings, some devices like smart TVs has it in their settings and some devices like routers has it enabled by default.

## Project

When I was a junior, I learned about this and made an example code in Python that receives a list of MACs and a list ofIPs and spams all combination possible to the router, project: [WakeOnLan](https://github.com/barakadax/WakeOnLan)

## Sources
- [Wake-On-Lan wikipedia](https://en.wikipedia.org/wiki/Wake-on-LAN)
