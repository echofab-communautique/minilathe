# minilathe# Minilathe

## Description
The JEULIN's MICROTOUR is a system developed initially for the education of the technoloy at College. There are 3 Minilathes at echoFab, but the interface and conection between them and the PC requires a specific software so the challenge was to make it work with Arduino and an open library like Grbl.

#### Specification
&nbsp;&nbsp;**Peak Height** : 100 mm  
&nbsp;&nbsp;**Distance between centers** : 400 mm  
&nbsp;&nbsp;**Permissible gross diameter** : 100 mm  
&nbsp;&nbsp;**Cone of the main spindle** : CM 3  
&nbsp;&nbsp;**Cone inside the shell** : CM 2  
&nbsp;&nbsp;**Section of the tool** : 12 mm x 12 mm  
&nbsp;&nbsp;**Longitudinal travel** : 300 mm  
&nbsp;&nbsp;**Transversal travel** : 90 mm  
&nbsp;&nbsp;**Power of the spindle motor** : 0,7 Kw  
&nbsp;&nbsp;**Resolution** : 0,01 mm  
&nbsp;&nbsp;**Machine weight** : 17 kg  
&nbsp;&nbsp;**Dimensions** : 98 x 67 x 53 cm  
&nbsp;&nbsp;**Controlled with** : Grbl & Arduino  
&nbsp;&nbsp;**Steps per revolution** : 200  
&nbsp;&nbsp;**Software** : Inkscape & Grbl  

## Development  
#### State of the machine  
The first step was to know the machine and how does it works since it hasn't been used in almost a year. The electric circuits and the mechanisms were still working, so I just had to know the outputs and the inputs of the system.

[[Fichier:IMG 20150217 162209892.jpg|right|150px]] [[Fichier:Minilathe driver.jpg|300px]]

The picture in the right shows the connection diagram of the driver, I tested all the inputs and outputs with a multimeter to make sure that everything was working and there were no connection errors.

[[Fichier:Arduino Uno.jpg|right|300px|caption|]]
#### Getting to know Arduino  
I have never used an Arduino, so it was necessary to learn the basics. Arduino is a microcontroller board that has digital and analogical inputs and outputs that can be connected to other circuits. It is very easy to use and the software is free and has a lot of libraries and examples to begin with, specially it had a step-motor command so I could use it to test the motors of the minilathe.






#### Testing the Machine  
Using a multimeter I was able to identify the inputs of each motor in the electric circuit of the machine so I was ready to start the first testing. With the Arduino step-motor command I was able to move the motors and do a first cut, also I found the sequence that they required to move.


[[Fichier:Firstcut.jpg|300px]]


#### Using Grbl  
Grbl is an open source g-code-parser and CNC milling controller that runs on Arduino. It has it's own libary and is very easy to use. You just download the library, then load it to the arduino and you can run Grbl, but you need Java installed first. Since Grbl has it's own library, it also has it's own outputs and inputs, and they were different from the ones that the minilathe required. The Arduino only sends 2 digital signals for each motor, one that is a pulse signal that controls the speed of the motor, and the other that controls the direction of the motor. The digital inputs of the minilathe were 4 for each step-motor, and were in a specific order that determinated the direction, and the speed was defined by the speed of the input changes. 


[[Fichier:Grbl pins.png|300px|caption|]]


For configuration of the settings and the interface you can visit their [[https://github.com/grbl/grbl/wiki|Wiki]]

#### Connecting Grbl and the Minilathe  
To be able to use the Arduino outputs and the Minilathe inputs I had to build a decoder, this is a device that converts binary information from a 2 pins input to a 4 pins input. Using a pic16f84a I programmed the decoder to convert the signal of the 2 outputs of the Arduino into 4 outputs that the minilathe could read. The grbl outputs were just one pulse that determined the velocity of the motor, and one binary signal that selected the direction, so the program I made was just a counter that depending on the direction gives a sequence of 4 outputs in the right order for the driver of the mini lathe to move.
[[Fichier:Pic and arduino.jpg|300px|caption|]]


#### First test  
With the decoder made I could use the interface of Grbl and made the first cuts using commands of G-code. After this first test I could calibrate the machine and repair some mistakes in the code that were provocating the heat of the motors, this was because the circuit of the minilathe inverts the signal of the inputs, so when I send a negative signal it was receiving a positive, making the motors to work in both directions at the same time overheating them.
[[Fichier:First test.jpg|300px|caption|]]

#### Inkscape  
Inkscape is a computer software program, which is used to draw and edit vector graphics. It is free and open source software. It has a library for exporting gcode from the vectors and a special tool for exporting gcode for the lathe. This was the reason why I decided to use Inkscape for the project but it was hard to understand the complete functioning of the software since there is no complete manual for the gcode tools and specially for the lathe gcode exporter.

First you have to define the tool you are going to use, you can do this in the tools library:

[[Fichier:Tools library.png|300px|caption|]]

In this text box you can define the parameters of your tool, the depth step depends on the material you are using and the process you are making, for fine cutting in wood I used 0.2mm because the wood I was working with was very fragile and I had to be very careful no to go too depth.
[[Fichier:Tool settings.png|400px|caption|]]

Once you have your tool defined you can make a gcode going to the lathe tool in gcode tools:

[[Fichier:Lathe.png|400px|caption|]]



For exporting a gcode you select the parameters of the piece, first you have the diameter of the blank piece (the one you are going to cut), then the depth of fine cut (the last path to be made so the piece has a smooth texture), the number of fine cuts to be made, the method to make the fine cut and the remapping of the axis. 

[[Fichier:Torno.png|400px|caption|]]



The diameter of the blank piece is represented as a radius so be careful not to put the actual diameter. 
The fine cut should be always bigger than the radius of your tool, otherwise it will just press the material instead of cutting it and make an annoying noise. 
The number of fine cuts depends on the tolerance your piece will have, but 1 or max 2 should be ok with most pieces.
Since grbl doesn't support the lathe gcode I remapped Z as the Y axis, this way grbl was able to reproduce the gcode without errors, if you use the Z axis it will make some strange spirals that mess up the piece. 
Once you have all your parameters adjusted you can click on apply and the program will generate a path like this:



[[Fichier:Corte.png|400px|caption|]]


I recommend to make a straight horizontal line above the piece and apply the lathe tool with the maximum depth to remove the material faster, but you can just make the piece since the beginning.

#### Final circuit
Finally I made a second shield so the arduino could communicate with the Mini-lathe:

[[Fichier:Second Shield.jpg|250px|caption|]]

The shield had an error connecting the 9th output of the arduino to the third input of the pic16f84a, it was supposed to be connecting the 8th and the third, so I had to solder and cut some tracks.

## User guide
#### Setting the Zero
For making a piece in the lathe you first have to set the 0 point, this have to match the 0 coordinates in Inkscape for the lathe to make the piece correctly. For setting the 0 point first you have to put the blank piece in the spindle and make sure that is centered so it will rotate without turbulences. Once the piece is correctly positioned in the spindle you can start the motor. With the piece spinning you go to the extreme of the piece and cut one millimeter, once you remove completely this millimeter of the piece you reset the zero in the machine control:



[[Fichier:machine.png|400px|]]

#### Importing the code
Once you have set the 0 point you just have to import the gcode by clicking on browse, and then click on send to start the machine, if you want you can see the process in the computer with the visualize button, but be careful because this is not just a simulation, the machine will start once you click "send". 
[[Fichier:imp.png|400px|]]


You have to be careful with the dimensions of the piece in inkscape and the dimensions of your blank piece in the lathe, and make sure your tool will make all the curves drawn, the minimum radius will be defined by your tool so it depends on the angle of your tool and the position.
[[Fichier:Angle.png|400px|]]
