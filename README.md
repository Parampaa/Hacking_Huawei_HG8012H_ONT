# Hacking Huawei HG8012H ONT
## Steps to hack a HG8012H, access it and mod the firmware


Due to the change from Vodafone to Masmóvil network that took place last year by Pepephone ISP, it was necessary to me to obtain an ADSL router compatible with the 802.1q protocol to be able to use the internet connection. This caused me to retire my Asus DSL-N55u router, with which I was very happy with its stability, configuration and WIFI coverage. So, I purchased a Huawei HG556a ADSL router and since this router had a 10/100 switch and pretty mediocre WIFI speed, I also purchased a Linksys EA8500 router that would provide me with the WIFI coverage and gigabit connectivity for my devices. So I configured the Huawei HG556a router in WAN Bridge mode and connected it to the WAN port of the Linksys router. This configuration gave me excellent performance, even more after flashing DD-WRT on the Linksys router.


Now, I have changed the internet operator, moving to a FTTH connection of 500/300 MB with Cableworld and they installed a Huawei HG8245U router and an optical splitter whose coaxial output, connected to home's TV splitter, distributes the TV channels of the service. As I was very happy with the operation and WIFI coverage of the Linksys Ea8500 router, I configured the HG8245U router again in WAN bridge mode and this is the way in which I have been working.

![GitHub Logo](https://github.com/logon84/Hacking_Huawei_HG8012H_ONT/blob/master/pics/1spliiter.jpg)


At this point several problems arise: The cableworld HUWAEI router is huge and I am lacking some space in the living room furniture, the use of two devices (HUAWEI router and optical splitter) forces me to use two power plugs, which increases the entanglement of cables in general. In addition, this router has a high power consumption, which is a waste of resources, since I am not using any of its ethernet outputs (except 1 connected to the Linksys WAN router), nor its Wi-Fi, nor 90% of its functions. So I started looking for a optical router/ONT with the slogan: small size, integrated CATV output, low power consumption and Huawei brand to be compatible with Cableworld's OLT. The perfect candidate was the Huawei HG8012H and in one of my trips to Portugal I was lucky enough to find one for € 5 in Cashconverters (normal price is about € 80).
