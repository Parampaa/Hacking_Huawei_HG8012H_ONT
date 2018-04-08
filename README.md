# Hacking Huawei HG8012H ONT
## Steps to hack a HG8012H, access it and mod the firmware


Due to the change from Vodafone to Masmóvil network that took place last year by Pepephone ISP, it was necessary to me to obtain an ADSL router compatible with the 802.1q protocol to be able to use the internet connection. This caused me to retire my Asus DSL-N55u router, with which I was very happy with its stability, configuration and WIFI coverage. So, I purchased a Huawei HG556a ADSL router and since this router had a 10/100 switch and pretty mediocre WIFI speed, I also purchased a Linksys EA8500 router that would provide me with the WIFI coverage and gigabit connectivity for my devices. So I configured the Huawei HG556a router in WAN Bridge mode and connected it to the WAN port of the Linksys router. This configuration gave me excellent performance, even more after flashing DD-WRT on the Linksys router.


Now, I have changed the internet operator, moving to a FTTH connection of 500/300 MB with Cableworld and they installed a Huawei HG8245U router and an optical splitter whose coaxial output, connected to home's TV splitter, distributes the TV channels of the service. As I was very happy with the operation and WIFI coverage of the Linksys Ea8500 router, I configured the HG8245U router again in WAN bridge mode and this is the way in which I have been working.

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/1spliiter.jpg)

At this point several problems arise: The cableworld HUWAEI router is huge and I am lacking some space in the living room furniture, the use of two devices (HUAWEI router and optical splitter) forces me to use two power plugs, which increases the entanglement of cables in general. In addition, this router has a high power consumption, which is a waste of resources, since I am not using any of its ethernet outputs (except 1 connected to the Linksys WAN router), nor its Wi-Fi, nor 90% of its functions. So I started looking for a optical router/ONT with the slogan: small size, integrated CATV output, low power consumption and Huawei brand to be compatible with Cableworld's OLT. The perfect candidate was the Huawei HG8012H and in one of my trips to Portugal I was lucky enough to find one for € 5 in Cashconverters (normal price is about € 80).


![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/2HG8012h.jpg)


### Hostility level 0

I got down to work configuring this ONT with my ISP parameters, so I connected an ethernet cable between the ONT and my PC and tried to open http://192.168.1.1 in the browser, but I got no response. Then I searched the internet for Huawei documents and technical files and found that for this model, the default IP address is 192.168.100.1 and the access users are telecomadmin:admintelecom and root:admin. After making the relevant IP and subnet changes in the network card of my PC, I went to http://192.168.100.1 in the browser and voila, the WebUi appeared to be able to start configuring my brand new ONT:

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/3login.png)

Unfortunately, none of the two access users I had found in the documentation worked. So it occurred to me to press the reset button for 30 seconds with a clip with the total certainty that this would cause the default access users to be functional again. But I was wrong. After a long time trying to search the internet for a login that would allow me access to the ONT, not only did I not find it but I verified that there is very little information about this device available. As if that were not enough, I discovered that when this ONT comes directly from an ISP, it is usually blocked by them to not allow access to the configuration and that way the user can not reuse it with another ISP. In this case, it appeared that the ONT that I bought in Cashconverters had not been purchased directly from HUAWEI but had been installed by a supplier. Things started to get complicated.


### Hostility level 1

I could see that by entering 3 incorrect passwords, the router did not allow retry the login until 1 minute after, so the option of dictionaries and brute force attacks to access was discarded by the amount of time the entire process would take.

Normally many routers allow telnet access to the device as an alternative way of configuration, but in this case it was impossible and the connection closed due to lack of response from the ONT.

Trying to find some weak point in the ONT that allowed me access, I did a port scanner from my PC, with the following command:

nmap -Pn -n -p0- 192.168.100.1


It did not help much, all ports were closed except port 80 (webUi). Port 23 (telnet) was not only not open but was being filtered by the integrated firewall to further complicate things. There was nothing else I could do externally to solve this, so screwdriver in hand I ventured to examine the bowels of the bug.

## Hostility level 2

Once the cover was opened and after a component identification phase, this is what I found:

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/4bottom_PCB.jpg)


![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/5spec.jpg)

Of all these components the ones that caught my attention were the serial port pads, the JTAG port and the flash memory. After a search on the internet, I deduced the pinout of both the serial port and the JTAG. They are quite common among HUAWEI router devices.

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/6jtag.png)

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/7serial.jpg)

I tested JTAG portusing the parallel port cable that can be seen in the previous photo but I did not get response, so I focused my efforts on the serial port. After connecting a TTL-USB converter and connecting to the virtual COM port using PutTy with a rate of 115200 symbols, I started to see the following bootlog on the screen:

> ggg
