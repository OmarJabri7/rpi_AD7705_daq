# Data acquisition with the AD7705 on the raspberry PI!

The AD7705 is a two channel sigma delta converter which has
differential inputs, a PGA and programmable data rates. It's
perfect for slowly changing inputs such as pressure, temperature,
heart rate etc.

This repo offers the class `AD7705Comm` which does all the low level
communications with the AD7705. The user just need to register a
callback handler which then returns the samples in realtime.

The class uses the DRDY of the AD7705 connected to Port 22 and
waits for a falling edge on this port to read
the data. This is done via interrupts / poll
so that the ADC process sleeps until new data becomes
available.

Further down I demonstrate how to use UDP to transmit the data to a web
page via PHP, JSON packets and client-side javascript!

## Hardware design

The design files (for EAGLE) are in the subdirectory "pcb".

![alt tag](circuit.png)

## Building the demo / test:

To build:

    cmake .

    make

## Running

Run it with the command:

    ./ad7705_test

## General usage

### Callback handler

```
class AD7705printSampleCallback : public AD7705callback {
	virtual void hasSample(int v) {
		// process your sample here
		printf("v = %d\n",v);
	}
};
```

### Example program

Instantiate the AD7705 class and the callback handler:
```
	AD7705Comm ad7705comm;
	AD7705printSampleCallback ad7705printSampleCallback;
	ad7705comm.setCallback(&ad7705printSampleCallback);
```

Start the data acquisition:
```
	ad7705comm.start(AD7705Comm::SAMPLING_RATE_50HZ);
```
once `start` has been called the data will be arriving.

Stop the data acquisition:
```
	ad7705comm.stop();
```

### UDP transmitter

The program `udp_transmit` sends the raw ADC values as ASCII strings
with a carriage retun at the end to port 65000. This program can
run in the background. Start it with:

```
./udp_transmit &
```
to run in the background.

To stop it when running in the background look up its process ID with
`ps ax` and then send HUP with the kill command:

```
kill -HUP <pid>
```

### UDP test receiver

For debugging purposes there is a python script which prints
the UDP packets on the screen: `UDP_receive.py`.

### Website which plots the UDP data

![alt tag](website/screenshot.png)

In the subdirectory `website` are examples how to plot the data (simple database
in a file and JSON packaging for realtime javascript animations). Copy these
files in the web-server directory `/var/www/html` of your PI. The preferred
web server is nginx. Please install it with the package manager
also the package `PHP`. Enable PHP for nginx.

## Author: Bernd Porr

bernd.porr@glasgow.ac.uk
mail@berndporr.me.uk
www.berndporr.me.uk
