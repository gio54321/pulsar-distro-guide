# Linux Pulsar Software for Recording and Analyzing
_package produced by Giorgio Dell'Immagine and Andrea Dell‘Immagine, IW5BHY_

_Summary by Hannes Fasching, OE5JFL oe5jfl(at)aon.at_

### Table of contents
* Installation of the distribution
* Recording and Conversion
* Analysis

## Installation of the distribution
* First of all get the link for downloading the ISO file (2GB) from Andrea or me (OE5JFL)
* make a bootable USB stick using any tool that permit it (`dd` if you are on linux or [rufus](https://rufus.akeo.ie/) if you are on windows)
* boot from the stick you have flashed changing the boot priority in the BIOS of your system (for most of modern PCs this is done automatically)
* once the bootloader (`grub`) shows up select the first option (`run linux`)
* if you want to install the OS onto your PC run `ubiquity` tool by double-clicking the `install` icon on the Desktop
* proceed with normal installation of Ubuntu OS (it should be easy but if you're struggling with it you can find tons of guides online like [this one](https://www.zdnet.com/article/ubuntu-15-10-a-walk-through-the-ubiquity-installer/))
* after the first boot open the terminal and the initialization will start; some links will be created on the Desktop
* only the first time run in the terminal
```
presto-makewisdom
```
and just wait (like a lot); this is a long test to optimize the performance of the FFT calculations.

## Testing out the SDR dongle
To test out the dongle you can run
```
gqrx
```
from the terminal; this is a basic SDR interface that should be fine if you want to test and experiment with gain, bandwidth and clearness of the band (it is usually good to make sure that there are no or minimum quantity of RFIs).

## Recording
For recording data from the dongle there is a nice graphical program. To start type in the terminal
```
rec-gui
```
and wait the tool to start. Choose the settings that you prefer and once you are happy with them start immediately the recording or set a specific time and let the countdown run. You can change the file name as well (actually you have to if you don't want to overwrite previous recordings) and it's location. Keep in mind that the `~/Desktop/PulsarData` folder is supposed to contain all the data files.
One example of filename could be
```
~/Desktop/PulsarData/b0329_jun08/bin0329_test.bin
```
Once the recording is started you should see the _I/Q cloud_ and the _power per channel_ graphs.

## Conversion
When the recording is complete you have to convert the _.bin_ file to a _.fil_ file. In order to do that there is another graphical tool to do it: just type in the terminal
```
bin2fil
```
and again set the values of the conversion so that they're coherent with the value of the recording. The filters value are calculated approximately by the formulas
```
FILT_H ~ 600/(pulse width of pulsar) (ms)
FILT_L ~ (pulse frequency)/20 up to (pulse frequency)/10
```
Tweak the `Ampli` value until you get a few % of saturation (shown in `% SAT`).

## Analyzing
If it's the _first time_ you analyze a file go to `Tempo` folder on Desktop and open the file `tz.in` with your editor of choice
* the first line contains information about the location of the radio telescope
* the last line is the name of the pulsar you want to calculate

Then go to the folder containing the _.fil_ file (b0329_test.bin) and open a terminal. Before the folding you should look for RFI in the recording. In order to do that you can run the following command
```
rfifind -o mask -time 1 -timesig 4 -freqsig 15 -chanfrac 1 -intfrac 1 b0329_jun08.fil
```
you can get information about these parameters with `rfifind -h`. During this process will be created some 'mask' files that indicates if and where are RFIs.

For calculating the pulsar period we need MJD time for the recording. We get this parameter by typing
```
header b0329_jun08.bin –tstart
```
In our case the result is `58227.02`. By the way the `header` command is a very useful tool to display information contained in te header of the _.fil_ file; the syntax is `header filename.fil`.

Now you need to calculate the polynomial curve that represent the change in time of the frequency. To do that type
```
tempo -z
```
and you will be asked for the first and last value of MJD; insert the value calculated previously. This will create some files, like `ployco.dat` or `tempo.lis`

To calculate the period of the pulsar you can use
```
polyco2period 58277.02 -p polyco.dat
```
this will output the period that you have to copy somewhere for further elaborations.

To fold the data use the command
```
prepfold -nsub 25 -n 500 -topo -dm 26.75 -nosearch -mask mask_rfifind.mask –p 0.714510274  -fine b0329_jun08.fil
```
again you can get informaions about these parameters with `prepfold -h`.

__If all went good you should finally see the result!__
