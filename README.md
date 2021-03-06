# EPrimeLabJack
Written by Gordon Matthewson, April 2015.

This is an E-Prime 2.0 function that allows E-Prime to send digital output signals from the Labjack U3 (labjack.com/u3).  This project was designed to allow the Labjack to replace a parallel port trigger system, as many modern desktops no longer support parallel ports.  Included is the E-Prime function TriggerLJ() that takes one or two decimal numbers, converts them into binary, and triggers FIO0-FIO7 (and EIO0-EIO7 on the CB15 if connected) with either a digital HIGH or LOW signal, depending on whether the digit is a 1 or a 0.  For example, TriggerLJ(49) would take the number 49, convert it to 00110001, output a HIGH signal on channels FIO2, FIO3, and FIO7, and a LOW signal on all the other channels.

This function requires the LabJack UD Windows driver, which is available here: http://labjack.com/support/software.  

The constants are taken from the UD module for Visual Basic (http://labjack.com/support/ud/examples/visual-basic): unnecessary functions have been stripped out for simplicity, but this would be where to look if you want to add functions to use the Labjack in E-Prime for something other than sending triggers (i.e. receiving them).

This package is designed for use with a single Labjack U3-LV and CB15 connector, although it will work if no CB15 is present.

The TriggerLJ function in this package showed a maximum latency of 8 ms when using an Empirisoft millisecond keyboard (http://www.empirisoft.com/directinkb.aspx) to trigger the function in E-Prime.  Timing was measured with a black box toolkit (blackboxtoolkit.com).  If your experiment requires millisecond precision, it would be best to measure latency yourself, as unique lab setups give rise to unique latencies. 

# How-to:
## Background
First, a quick note about how E-Prime actually works with the LabJack.  This section can be used for reference: if you’d like to skip to how to use the E-Prime functions in your experiment, go on to the "function" section below.

Within E-Prime, commands to the LabJack itself are made by setting the variable “lngError” equal to a particular LabJack driver command.  For example, within the InitLabJack function,  

lngError = OpenLabJack(LJ_dtU3, LJ_ctUSB, “1”, 0, lngHandle)

opens the first found Labjack U3 over USB.  The command “OpenLabJack” is called from the .dll file that is installed with your driver.  If you wanted to expand functionality of your LabJack and use commands from the driver not included in this package, remember to first declare them in E-Prime, either by adding Declare functions in the User Script of your experiment, or by modifying this package file and adding them there.

“lngError” is used for debugging: when anything other than 0 is returned, an error is raised that corresponds to a particular constant defined in this package, with 50,000 added to it so that E-Prime knows it is a “user-defined” error.  For example, if your LabJack is not plugged in, running the above command would raise the error:

Application-defined or object-defined error

Line: 001
Error Number: 51007

Which would correspond to the global constant “LJE_LABJACK_NOT_FOUND”, as this is set to 1007 under the “error codes” section of the package.

## Functions

This package contains two E-Prime functions: InitLabJack, and TriggerLJ().  Copy and paste the "EPrimeLabJack" text into the user script of E-Prime (you can get to this by pressing Alt+5).

InitLabJack does three things: 

1. Opens the first found LabJack U3 over USB
2. Sets all pins to a factory default condition
3. Sets all ports (including the EIO ports, if you have a CB15) as digital outputs

This function is defined in the user script, but it needs to be initialized to actually perform these operations.  Adding

InitLabJack

to an InLine script at the beginning of your experiment will call this function and get your LabJack ready to go.

Assuming there were no problems with InitLabJack, your LabJack is ready to send digital messages.  This can be achieved by calling the function:

lngError = TriggerLJ(Trigger1, Trigger2 (optional), “rev_channel” (optional))

Remember here that the actual function is TriggerLJ(), but you must preface it with an lngError = for it to work.  You don't need to write this bit before InitLabJack, just the subsequent TriggerLJ() function.

This function takes up to three parameters: 

* Trigger1: A decimal number which will be converted to binary and trigger FIO0-FIO7
* Trigger2 (optional): A decimal number which will be converted to binary and trigger EIO0-EIO7 on the CB15
* “rev_channel” (optional): This parameter was added because the machinery in our particular lab (Medoc Pathways thermode)is from Israel, where numbers are read right to left, thus creating a need for us to reverse our messages to this machine.  Remember to specify this input as a string by putting parentheses around it.
	* "rev_channel_a” : will reverse your first number (00110001 would become 10001100)
	* “rev_channel_b” : will reverse your second number
	* “rev_channel_ab” : will reverse both numbers

# Example Usage:

lngError = TriggerLJ(49)

will send binary 49 to ports FIO0-FIO7,

lngError = TriggerLJ(49, “rev_channel_a”)

will reverse this signal, and 

lngError = TriggerLJ(49, 100, “reverse_channel_ab”)

will send a reversed binary 49 to FIO0-FIO7, and a reversed binary 100 to EIO0-EIO7.

Each time you send a message with this function, follow it with a 

Sleep(100)

command, followed by a 

lngError = TriggerLJ(0,0)

command, which will reset all ports back to an OUTPUT-LOW status.

# Our setup

This is what our setup looks like.
![Labjacksetup](/lj.jpg)
Note: there are probably almost twice as many wires as there need to be: this is because the wire that was laying around our department just happened to have a ground for each wire.  I think you really only need one ground per parallel port, but from what I've heard wrapping each wire with a ground helps protect the integrity of the signal over long distances.

### Parallel Port 1
Labjack Port | Parallel Port Pin
----------- | -----------------
GND  | Pins 18-25
FIO0 | Pin 2
FIO1 | Pin 3
FIO2 | Pin 4
FIO3 | Pin 5
FIO4 | Pin 6
FIO5 | Pin 7
FIO6 | Pin 8
FIO7 | Pin 9

### Parallel Port 2
Labjack Port | Parallel Port Pin
----------- | -----------------
GND  | Pins 18-25
EIO0 | Pin 2
EIO1 | Pin 3
EIO2 | Pin 4
EIO3 | Pin 5
EIO4 | Pin 6
EIO5 | Pin 7
EIO6 | Pin 8
EIO7 | Pin 9

This is what the other end looks like.
![Labjacksetup](/LJ2.jpg)
There is heat-shrink plastic over each of the ends (except for a few that I forgot).  This is an empty parallel port hub that I soldered each wire into: digital outputs into the standard parallel port data pins (2-9), and their grounds into standard parallel port ground pins (18-25).
