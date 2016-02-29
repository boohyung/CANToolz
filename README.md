# CANSploit

CANSploit is a framework for analysing CAN networks and devices.
This tool based on different modules which can be assembled in pipe together and
can be used for security researchers and testers for black-box analyses.

## Using a Hardware

CANSploit can work with CAN network by using next harware:

1. [USBtin](http://www.fischl.de/usbtin/)
2. [CANBus Triple](https://canb.us/)

## Modules

- hw_CANBusTriple  - IO module for CANBus Triple HW
- hw_USBtin.py	 - IO module forUSBtin
- mod_firewall     - module for blocking CAN message by ID
- mod_fuzz1        - Simple 'Proxy' fuzzer  (1 byte)
- mod_printMessage - printing CAN messages
- mod_uniqPrint    - CAN messages statistic (with .csv file output)
- gen_ping.py	     - generating CAN messages with chosen IDs (ECU/Service descovery)

P.S. of course we are working on supporting other types of I/O hardware and modules. Please join us!
Main idea that community can produce different modules that can be usefull for all of us 8)

## Dependencies

    pip install pyserial


# Usage Examples
   See more use-cases inside examples folder:
        - MITM with firewall
        - replay discovery
        - ping discovery
## Simple MITM with CAN firewall

1. Put config file `can_sploit.cfg`

        [hw_USBtin]
        port = auto         ; Serial port
        debug = 1           ; debug level (default 0)
        speed = 500         ; bus speed (500, 1000)

        [hw_USBtin~2]
        port = auto         ; Serial port
        debug = 1           ; debug level (default 0)
        speed = 500         ; bus speed (500, 1000)
        bus  = 62

        [mod_firewall]

        ; Sequence of modules, logic of MITM/FUZZ and etc
        [MITM]

        hw_USBtin = {'action':'read','pipe':2}            ; read from first HW module to PIPE number 2
        hw_USBtin~2 = {'action':'read'}                  ; read from second HW module to PIPE number 1 (default is 1)
        mod_firewall = {'white_list':[112,113],'pipe':2} ;block messages with ID 112,113 from PIPE number 2
        hw_USBtin~2!2 = {'action':'write','pipe':2}  ; write via second HW module messages from PIPE number 2
        hw_USBtin!2 =  {'action':'write','pipe':1} ; write via first HW module messages from PIPE number 1 (not firewalled)

This config make possible full MITM with with help of two USBtin devices.


P.S CANBus Triple support up to 3 CAN buses, so with that hardware it will be enough only one IO module! (but speed is lower than USBtin)

	~ - used in config if we want to load same module more than once
	! - used in config if we want to use same module more than once (temporary hack, because Pyhton config parser can't use same directive twice)

2. run CANSPloit

        python cansploit.py -c can_sploit.cfg
        hw_USBtin: Init phase started...
        hw_USBtin: Port found: COM14
        hw_USBtin: PORT: COM14
        hw_USBtin: Speed: 500
        hw_USBtin: USBtin device found!
        hw_USBtin: Init phase started...
        hw_USBtin: Error opening port: COM14
        hw_USBtin: Port found: COM13
        hw_USBtin: PORT: COM13
        hw_USBtin: Speed: 500
        hw_USBtin: USBtin device found!
        >> h

        start           Start sniff/mitm/fuzz on CAN buses
        stop or ctrl+c  Stop sniff/mitm/fuzz
        view            List of loaded MiTM modules
        edit <module> [params]          Edit parameters for modules
        cmd <cmd>               Send some cmd to modules
        help            This menu
        quit            Close port and exit

        >> start
        >>

        Add new ID to the blocking list on the fly:

        >> stop
        >> v
        Loaded queue of modules:

        Module hw_USBtin: {'action': 'read', 'pipe': 2}
                ||
                ||
                \/
        Module hw_USBtin~2: {'action': 'read', 'pipe': 1}
                ||
                ||
                \/
        Module mod_firewall: {'pipe': 2, 'white_list': [112, 113]}
                ||
                ||
                \/
        Module hw_USBtin~2!2: {'action': 'write', 'pipe': 2}
                ||
                ||
                \/
        Module hw_USBtin!2: {'action': 'write', 'pipe': 1}
        >> e mod_firewall {'white_list': [112, 113,114,115],'pipe':2}
        Edited module: mod_firewall
        Added  params: {'pipe': 2, 'white_list': [112, 113, 114, 115]}
        >> v
        Loaded queue of modules:

        Module hw_USBtin: {'action': 'read', 'pipe': 2}
                ||
                ||
                \/
        Module hw_USBtin~2: {'action': 'read', 'pipe': 1}
                ||
                ||
                \/
        Module mod_firewall: {'pipe': 2, 'white_list': [112, 113, 114, 115]}
                ||
                ||
                \/
        Module hw_USBtin~2!2: {'action': 'write', 'pipe': 2}
                ||
                ||
                \/
        Module hw_USBtin!2: {'action': 'write', 'pipe': 1}
        >> s

That's it for now. Will update this file later with more config, and examples how to use this tool!


Alexey Sintsov
alex.sintsov@gmail.com
