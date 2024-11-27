### Side channel attack

Capturing RF signals can give out information about what the device is currently doing, and computer monitors are no exception for this issue. Using a software called TempestSDR, it is possible to recreate a black and white live image of what a screen is displaying thanks to the RF signals that the HDMI cable is leaking. After calibrating the frequency at which the signals are being leaked and tinkering with the settings that TempestSDR provide, we have a clear image of what our display is showing!

![sdrsharp_calibrar](https://github.com/user-attachments/assets/759574c3-164b-4c49-96a7-e9b5e7fd59d3)
_*Figure 1: RF signals being leaked from an HDMI cable shown in SDR#_


![jtsec_notepad](https://github.com/user-attachments/assets/9e8ef603-7dd4-49ed-9b2a-d88aeda6a06b)
_*Figure 2: Display of the victim_

![tempestsdr](https://github.com/user-attachments/assets/c57fdb86-9e50-49ef-9a3c-88f3685c5523)
_*Figure 3: Recreated picture shown in TempestSDR_

#### Needed software
The HackRF One can be used in both Windows and Linux with widely available and well-maintained software. In the windows side of things, all that is needed to get started is is SDR#.

![sdrsharp_calibrar (1)](https://github.com/user-attachments/assets/9121cb59-04e3-468b-8206-5a1de067dc6c)
_*Figure 4: SDR# Graphical User Interface_

In Linux, installing the driver packages is necessary to get the tools to communicate with the HackRF One. This can be done in Debian based distributions by simply installing the "hackrf" package. Now to use the device to receive and record RF signals, Gqrx is the open-source SDR# equivalent in Linux and it is also present in the Debian repos.

![gqrx](https://github.com/user-attachments/assets/3f77fca2-2478-4ab8-ad11-24c363c96380)
_*Figure 5: Gqrx Graphical User Interface_

These tools are pretty similar in what they do, both allow the user to monitor the RF spectrum, receive RF signals at a certain frequency and demodulate them in different modes like FM, AM or USB, making it possible to recover the information that is being transferred like the audio being broadcasted from a radio station. Both also present a similar layout where the top half is used to display the RF spectrum (signal strength for a given frequency at a point in time) and the bottom one used to display a waterfall plot representing that signal strength over time. Given that the HackRF is also able to transmit, it is possible to record and replay these signals over RF.

Going back to the main goal behind SDR devices, we talked about the idea of being able to receive and transmit different RF protocols with a single device just by configuring its software. Well, the programs that we just talked about only allow a set of predefined actions and configurations to be performed but, What about interacting with radio frequencies in a more programming-oriented way? This is where GNU-radio comes in. GNU radio is an Open Source SDK that provides signal processing blocks that can be combined and interconnected in a graphical manner thus simplifying creating programs that work with RF.

![gnuradio](https://github.com/user-attachments/assets/425c1bbe-fc10-4b4e-a465-9680e90398f8)
_*Figure 6: GNU Radio project example to capture Wifi packets_
