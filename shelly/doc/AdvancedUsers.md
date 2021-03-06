# Advanced Users

This section provides information for advanced use cases.

## Additiona Resources

There are additional resources available providing more information on Shelly devices and how to integrate those into openHAB:

- [Shelly Homepage](https://shelly.cloud)
- [Shelly Support Group (English)](https://www.facebook.com/groups/ShellyIoTCommunitySupport)
- [Firmware Archive](http://archive.shelly-faq.de)
- [API Documentation](https://shelly-api-docs.shelly.cloud/?fbclid=IwAR23ukCi_3aBSTPRHYUIcpr0pLi0vcyL0fF0PnJQdFvkkc8_Zo5LkAcli_A#http-server)

and the [openHAB Community thread specifically for the binding](https://community.openhab.org/t/shelly-binding) - any feedback welcome here.
You could also [report a bug or request a feature](https://github.com/openhab/openhab-addons/issues?q=is%3Aissue+is%3Aopen+%5Bshelly%5D) for the Shelly Binding.

## Firmware Upgrade

The Shelly App usually displays the installed firmware and also provide the function to upgrade the device with new firmware.
However, if this doesn't work (sometimes there are issues) you could use the [Shelly Firmware Archi Link Generator](http://archive.shelly-faq.de), which provides download links to current, but also archived firmware files for all devices. 

|Version|Notes                                                                                             |
|-------|--------------------------------------------------------------------------------------------------|
|1.5.7  |Minimum supported version. Older versions work in general, but have impacts to functionality (e.g. no events for battery powered devices). The binding displays a WARNING in the log if the firmware is older.|
|1.6.x  |First stable CoIoT implementation. AutoCoIoT is enabled when firmware version >= 1.6 is detected. |
|1.7.x  |Add additional status update values, fixes various issues                                         |
|1.8.0  |Brings CoIoT version 2, which fixes a lot issues and gaps of version 1.                           |
|1.9.2  |Various improvements, roller favorites, CoAP fixes                                                |


There are 3 options available to perform the upgrade
- The Shelly App usually detects when a new version becomes available and offers to do the upgrade within the UI (Web or App)
- Alterco provides the [Shelly Firmware Archive Link Generator](http://archive.shelly-faq.de).
This can be used to generate the upgrade link, which could be easily used to perform the upgrade on the cli-level having an Internet connection on that terminal (Shelly device doesn't require an Internet access).
You specify the device's IP and device model SHSW-25 and the page will generate you the link for the firmware download using the OTA of the device.
Then you run "curl -s [-u user:password] &gt;generated link&gt;" from the terminal.
This should show a JSON result, make sure that it shows "status:updating".
Wait 15sec and access the device's Web UI, go to Settings:Firmware Upgrade and make sure than the new version was installed successful.
- Manual download and installation of the firmware
Manually pick the download link from the [Shelly Firmware Repository](https://api.shelly.cloud/files/firmware) and get the release or beta link.
Once you downloaded the file you need to copy it to an http server. 
Open the following url http://&lt;shelly ip&gt;/ota?url=http://&lt;web server&gt;/&lt;path&gt;/&lt;zip-file&gt;
Again, make sure that the file is downloaded and installed properly.

## Trouble Shooting

### Network Settings

Shelly devices do only support IPv4. 
This implies that the openHAB host system has IPv4 bound to the network interface.
The binding is only able to discover devices on the local subnet. 
Add things manually with the given IP if you have a routed network in between or using a VPN connection.

The binding enables CoIoT protocol by default if the device is running firmware 1.6 or newer.
CoIoT is based on CoAP and uses a UDP based signaling using IP Multicast (224.0.1.187, port 5683).
Again if the device is not on the same local IP subnet you need special router/switch configurations to utilized CoAP via IP Multicast.
Otherwise disable the Auto-CoIoT feature in the binding config (not the thing config), disable CoIoT events in the thing configuration and enable sensors events (http callback).
Nevertheless in this setup the binding can communicate the device, but you are loosing the benefits of CoIoT.

Refer to openHAB's general documentation when running openHAB in a docker container. Enabling mDNS discovery has additional setup requirements.  

### Re-discover when IP address has changed
 
Important: The IP address should not be changed after the device is added to openHAB.

This can be achieved by

- assigning a static IP address (recommended for battery powered devices) or
- using DHCP and setup the router to always assign the same IP address to the device

When the IP address changes for a device you need to delete the Thing and then re-discover the device.
In this case channel linkage gets lost and you need to re-link the channels/items.

## Log optimization

The binding provides channels (e.g. heartBeat, currentWatts), which might cause a lot of log output, especially when having multiple dozen Shellys.

openHAB has an integrated feature to filter the event log.
This mechanism doesn't filter the event, but the log output (items still receive the updates).

A configuration is added as a new section to `openhab2-userdata/etc/org.ops4j.pax.logging.cfg`

Based on this you could use the following config:

```
# custom filtering rules
log4j2.appender.event.filter.uselessevents.type = RegexFilter
log4j2.appender.event.filter.uselessevents.regex = .*(heartBeat|LastUpdate|lastUpdate|LetzteAktualisierung|Uptime|Laufzeit|ZuletztGesehen).*
log4j2.appender.event.filter.uselessevents.onMatch = DENY
log4j2.appender.event.filter.uselessevents.onMisMatch = NEUTRAL
```

This filters events for items heartBeat, lastUpdate, LetzteAktualisierung, Uptime, Laufzeit, ZuletztGesehen. Replace those strings with the items you want to filter. Use a list of items to reduce logging.
Please note: Once events are filtered they are not show anymore in the logfile, you can’t find them later.
