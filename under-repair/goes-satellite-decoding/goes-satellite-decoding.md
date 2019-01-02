# GOES Satellite Decoding

Back in February of this year I found a fantastic site, [SetiQuest](http://setiquest.org). Like a lot of people, I’ve been fascinated by the idea of SETI for a long time and to have the opportunity to play around with their data was great. The actual process of searching these signals for signs of intelligent life doesn’t interest me too much, though the matched filtering methods proposed by [Professor David Messerschmitt](https://people.eecs.berkeley.edu/~messer/) seems very interesting. What does interest me is the calibration data collected from satellites. This data often contains information bearing signals that can be demodulated and decoded to give a wide variety of information. The data set that most interested me was a recording made while targeting a geostationary operational environmental satellite (GOES) satellite. These satellites provide constant monitoring of weather related events on earth and beam down a variety of images and text information.

With the data sets provided I was able to decode a number of high quality images from both GOES-East (aka GOES-13) and GOES-West (aka GOES-11). GOES-East is stationed above the northern part of South America and is able to see both the North and South American continents as well as the eastern coast of Africa. The satellites cycle through a series of imaging sectors targeting different parts of the globe with capturing various segments of the RF spectrum. The image below is of the continental United States taken in the visible light spectrum from GOES-East.

![GOES East 13 North America](TBD)

GOES-West is stationed over the Pacific Ocean and can view the continental US as well as Alaska and Hawaii. The image shown below targeted the south hemisphere in the visible light spectrum.

![GOES West 11 South Pacific](TBD)

When I originally did this decoding, I used Matlab for the entire process. I’ve since rewritten everything for C++ to speed things along, but I thought I would detail my original demodulation and decoding process in case anyone is interested.

## SetiQuest Data Formatting

The first step in the decoding process is understanding how the data is saved by SetiQuest. The SETI website gives a nice description of how they collect and save their data so I’ll just give a brief overview. The data is saved in a series of .dat files which contain 8 bit complex numbers corresponding to the IQ demodulated data. Each data set on the SETI website comes with a header file which outlines the center frequency the data was collected at, along with the bandwidth of the data.

For the GOES data sets, the center frequency was 1692 MHz with a bandwidth of ~8.74 MHz. You can see an amplitude plot of the spectrum below.

![Geostationary Operational Environmental Satellites GOES Spectrum](TBD)

I’ve labelled the locations of some of the signals transmitted down by the GOES satellite. At 1691 MHz is the Low Rate Information Transmission (LRIT) signal. This is the signal which contains the information for the above images. At 1692.7 MHz is the [EMWIN-N](http://www.nws.noaa.gov/emwin/) signal. At 1694 MHz there is a low data rate telemetry signal. Between 1694 MHz and 1695 is a Data Collection Platform Report (DCPR) signal. I’ve been primarily concerned with the LRIT signal since this is the most well documented signal, however I have briefly looked at the other signals. The telemetry signal appears to be a 4000 bps, manchester encoded signal. I couldn’t find any sources that provide the details of this signal so I didn’t explore it too far. I hope to spend some time looking at the EMWIN-N signal, I believe this signal is modulated with an offset quadrature phase-shift keying approach.

## LRIT Demodulation

The demodulation and decoding of the LRIT is fairly straightforward. The only real complexity comes in the number of signal processing steps that lie between the raw data and the final image. My plan is to break the entire demodulation process into a series of posts. I’ll post as much processed data from each step as possible. To get the original raw data, you need to sign up at the setiQuest website and then you can download the data from [here](http://setiquest.org/getting-data).

For this post, I’ll focus on bringing the passband signal down to baseband and sampling the bits. There is a nice [report](/http://www.goes-r.gov/hrit_emwin/ATR-2010-5482.pdf) from the Aerospace Corporation which details the transmission parameters of LRIT as well as some of the other signals transmitted by the GOES satellites. Below you can see a table from that report which summarizes the transmission specifications very nicely.

![Aerospace Corporation LRIT Specs](TODO Add path)

The first step in the demodulation process is filtering the signal to isolate the LRIT signal. Since this is a complex signal, we can not simply use a real-valued bandpass filter. A real bandpass filter is designed to pass both positive and negative frequencies, but in the case of this data, the negative frequencies are not just a complex conjugated copy of the positive frequencies. The bandpass filter was created by starting with a lowpass filter with a bandwidth equal to a half my desired bandpass bandwidth (~350 KHz lowpass bandwidth). I then multiplied the real-valued lowpass filter with a complex sinusoid whose discrete frequency was equal to the discrete frequency of the LRIT signal, this frequency is 2*pi*(1691e6 – 1692e6)/sF, where sF is the sampling frequency. The graphic below shows the filter superimposed on the LRIT signal.

![LRIT Spectrum and Filter Passband](TBD)

The next step is the carrier recovery to bring the BPSK signal to baseband. Because no two systems have exactly synchronous clocks, you can’t simply shift the passband signal to baseband with a complex sine multiplication. Instead, I use a Costas loop to track the carrier, and use this recovered carrier to bring the system to baseband. The Costas loop is essentially a PI controller for the frequency of the synthesized carrier. The error fed into this controller is generated by multiplying the real part of the baseband signal by the imaginary part. This creates a signal which is proportional to the difference in frequency between the received carrier and the synthesized carrier. Often times you will see this error term passed through a low pass filter, but since we are dealing with a complex passband signal this isn’t strictly necessary. If the passband signal is a real signal, there will be harmonics at 2 and 4 times the carrier frequency which need to be filtered out. The proportional term of the Costas loop will cancel any frequency offset, while the integral term will bring the phase offset to zero. Below you can see the baseband signal in blue and the sampling times for each bit marked with the black star.

![LRIT BPSK Baseband Signal and Timing](TBD)

The timing recovery was performed using a Mueller-Muller timing scheme. This bit timing recovery uses the current sample along with the previous sample to determine if the sampling instance is early or late. If the sampling is early, the symbol timing period is increased, if the timing is late, the symbol period is decreased. You can see the constellation diagram for the BPSK signal below.

![LRIT BPSK Constellation Diagram](TBD)

I’ve collected all the mFiles I used to get the above data along with a text file containing the first 2 million bits from the GOES-13 second data set. If you’d like to run the files yourself, you should only need to change the file location of the GOES data set. You can download the scripts and data [here](TBD/GOESpost_01.zip). In the next few days I plan to discuss the Viterbi decoding, packet synchronization and de-randomization.
