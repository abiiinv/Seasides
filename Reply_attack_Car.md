#### Target: We’ll be attacking an Opel Astra car, the only car I was close to at the time of writing.
![Opel_Astra_G_front_20081128](https://github.com/user-attachments/assets/41474907-56cc-4951-99e6-5b1e82bb4453)

#### Gear: HackRF One
![hackrf_one_bg](https://github.com/user-attachments/assets/392b7b3b-ac0f-452a-9bf0-1094fd33c7ef)


HackRF One is a SDR device which can transmit and receive radio signals in the range of 1 MHz up to 6 GHz. It was designed to enable test and development of modern and next generation radio technologies. It’s an open source hardware platform that can be used as a USB peripheral or programmed for stand-alone operation.

We’ll be using it to receive and transmit signal.

#### Determining the frequency
First and most important, frequency on which the key operates needs to be determined. For that, CubicSDR can be used, after connecting a HackRF to the computer running it.
![carfreq](https://github.com/user-attachments/assets/4bd5ff3f-c006-4640-a7af-bbb7e0e2e701)

After opening up the CubicSDR software, the frequencies between 433 and 434 MHz can be scanned, which are the most likely frequency for it to be operating on, based on some quick online research. Even if there is no clue whatsoever what the frequency is, the search range can be broadened and zoomed into the first signal that corresponds to the timing of pressing the keys on the RF controller. The frequency found is around 433.9 MHz, and since each SDR receiver has its own margin of error (many software programs use frequency correction to deal with this), it’s safe to assume that it’s indeed using the standard 433.92 MHz frequency that most car keys operate on. It can also easily be seen that the HackRF has no trouble finding the signal the car key sends

#### Recording the signal
Now that the frequency the car key operates on is determined, it’s possible to intercept the signal using GNURadio Companion and record it to a file, so it can be replayed later.
![signalrecord](https://github.com/user-attachments/assets/d7e6dae2-96bb-4483-aa6f-b92d66c464c7)

As seen in the picture above, the flow graph is really simple. On the left side there is a variable block that determines the sample rate used, 10 million samples per second, and an osmocom Source block. Osmocom blocks are primarily developed for OsmoSDR hardware by Osmocom, but they in fact support a wide range of devices, including the HackRF through the libhackrf library, made by the creator of HackRF, Michael Ossmann. The blocks enable interaction with the HackRF device through the GNURadio software. This particular block, the Source block, enables users to tune in to the device with the settings they want and receive and manipulate the data it receives based on the settings they give it. On the right side there is a QT GUI Time Sink, which enables visualising the received data, and if necessary to make some adjustments to the settings of the source block based on the data. Last, there is the File Sink which streams the signal from the source directly to a file on the computer running GNURadio. Since the osmocom Source block is the most important part of the flowgraph here, each and every one of the settings on it will be analysed, explaining what they are used for. They will also be of use when describing the flowgraph used for replaying the attack.



- Clock Source defines the source of the timer the device should use, and it’s set to use its own internal clock.
- Sample rate’s name speaks for itself, and it uses the predefined samp_rate variable.
- Frequency(Hz) is set to 433.92 MHz that was defined earlier.
- Frequency correction is off, since the HackRF’s small offset is tolerable. Otherwise it would be necessary to input the correction using a parts per million (PPM) value, which is unique to every SDR receiver.
- Due to the usage of direct conversion (homodyne or zero-IF) recievers, which are very popular today, problems like a huge spike in the center of the spectrum (DC offset) and a phase and/or amplitude imbalance between In-phase and quadrature signals (IQ), it’s needed to employ certain corrections. These, while being a very broad and interesting topic for themselves, ultimately have no effect on the setup of this attack, so the DC offset and IQ balance mode will be left turned off.
- Last, but not least, therea re the gain settings. The gain mode is set to manual, so the amount of gain to be used can be manually provided (in decibels). The Radio Band (RF) gain is off, since it won’t be necessary, but the Base Band (BB) and Intermediate Band (IF) gains are on, and set to 16 decibels each. There really isn’t any magical number to determine the amount of gain needed, since it depends on the strength of the signal source and our proximity to it. But to be a fair user of the radio spectrum, which many people use and depend on, minimal amount possible should be used, not to interfere with other users or to unnecessarily ‘sniff’ their traffic. Small amounts of gain can be started with, and if necessary, they can be turned gradually up (not the other way around!). But not to write every iteration possible, the final amount of gain used in this attack is 16dBi for the receiving part.
![guitimesinkrecord](https://github.com/user-attachments/assets/a3f5193e-3deb-4bdd-9935-b8d20e3bcb8a)

Upon execution of the flowgraph, and pressing the unlock button on the car key, the waveform of the signal the key sends can be seen. After stopping the flowgraph, a file should be stored on the computer running GNURadio, containing the signal. Because of the high sample rate, the file can easily reach sizes up to 1GB for a few seconds of recording.

#### Analysing and replaying the signal
After recording the signal, we’re ready to begin the replay attack phase. First, a flowgraph for replaying the signal has to be made, and the signal quality needs to be checked, to determine if it is good enough to be reproduced reliably.
![signalreplay](https://github.com/user-attachments/assets/d305de5f-e095-4450-9adb-0b7431d166df)

The flowgraph for replaying the signal is also fairly simple, the difference is that this one is using an osmocom Sink block to replay the data through the HackRF, and a Throttle block, since the visualisation of the raw data with the QT Sink would be very CPU intensive. After trying to replay the captured signal, nothing happens. The attack doesn’t work. The reason why can be seen through the QT GUI Time Sink.
![guitimesinkreplay](https://github.com/user-attachments/assets/add32023-f6e2-47df-9089-b7dff4da79b1)

It can easily be seen that the maximum and minimum amplitude reached is only approximately 0.35, which isn’t that great considering it can go up to 1 and down to -1. So, the signal needs to be amplified to at least be doubled somehow, to reach 0.7 amplitude or more, and GNURadio has just the block for that.

The block is called Multiply Constant, and it can be used it to multiply the captured signal by any given number. Since the goal is to at least double the signal amplitude, 2 can be used as the constant.
![guitimesinkreplay2](https://github.com/user-attachments/assets/b8f2ff8c-5458-40b0-9286-02fa038abce8)

Upon execution of the replay attack, the difference in the amplitude of the signal we’re sending to the car is immediately seen, and sure enough, the attack works. The car can now be locked or unlocked by executing the flowgraph given the file containing the saved signal.

