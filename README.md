# Emergency Alert App Using Mamdani Fuzzy Logic

This repository contains simulink files that implement an Emergency Alert Android App. The app sends device's GPS location to another nearby device using Bluetooth on detection of emergency. Two methods have been discussed for detecting the level of emergency based on surrounding audio level and acceleration of the device. 

**See the demo video [here](https://www.youtube.com/watch?v=-zVNOG2WCnU&list=PLoLZwK9ryVfe_2H1-wBAL3OWbL5Du3fRt).**

**MATLAB File Exchange - [here](https://in.mathworks.com/matlabcentral/fileexchange/102915-mamdani-fuzzy-logic-app)**


Applications in real life

- If a person is in an emergency situation, they can shout out load and shake their device to send emergency alert to another person.
- In case of a car accident, the noise level could increase and acceleration can also increase suddenly. Such a situation can be detected by the app.
- There can be several other scenarios where emergency can be detected based on surrouding noise level and sudden jerk / free fall  (acceleration) of device.

## Installation Requirements

- [Matlab Simulink](https://in.mathworks.com/products/simulink.html)
- [Fuzzy Logic Toolbox](https://in.mathworks.com/help/fuzzy/referencelist.html?type=block&category=getting-started-with-fuzzy-logic-toolbox&s_tid=CRUX_topnav)


## Problem Statement

Designing an emergency alert app using the accelerometer and the microphone signal from mobile phone. The degree of emergency should vary according to the changes in acceleration and the changes in microphone signal amplitude. If the highest emergency level is detected, send the device's GPS location to a nearby device using Bluetooth.  

- (Part a) Use basic logic blocks to implement the app.

- (Part b) Use Mamdani fuzzy logic to detect the emergency using suitable logical rules.

## Implemented Solution

### (Part a) Using Normal Logic

#### Files

- [`Part_a.slx`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/Part_a.slx)
- [`Receiver.slx`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/Receiver.slx)

#### Taking Input

- Acceleration

    It is taken from device's sensor and has three values - X, Y, and Z axis. To convert it into a single value, we take the sum of absolute of these three values.

    e.g. `X-value: 3, Y-value: -2, Z-value: -1`, then `final value: 6`

- Audio Amplitude

    It is also taken from device's sensor and it is a matrix that contains positive and negetive values. We taken the value with the maximum absolute value.


#### Sending GPS Location


**This was the most challenging part**. Key challenges were -

- Setting up Andriod apps on both devices and using appropriate service and characteristic for Bluetooth. This was solved by following the  [documentation](https://in.mathworks.com/help/supportpkg/android/ref/work-with-ble-blocks-on-android-devices.html) from Mathworks.
- The sender device keeps sending (0, 0, 0) as Lat, Long, Alt GPS location to the receiver device when there is no emergency, and when it detects emergency it sends the actual GPS location. 
- Here, the problem is once the emergency level is detected (i.e. it becomes 1), GPS location is sent to receiver device but in the next clock tick the emergency signal would become 0. Hence, on the receiver device the GPS location would be shown only for fraction of time.
- To resolve this, [persistant](https://in.mathworks.com/help/matlab/ref/persistent.html?searchHighlight=persistent&s_tid=srchtitle_persistent_1) keyword was used to build a state machine on sender device. 
- Whenever a new emergency is detected, the current new GPS location is sent from BLE Send otherwise the sender device keeps sending previously detected GPS location (when previous emergency was detected). If no new emergency is detected, the sender device keeps sending current GPS location and it is shown persistently in receiver device. Initially (0, 0, 0) is sent if no emergency has been detected.



#### Core Logic

The file [`Part_a.slx`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/Part_a.slx) contains simulink blocks for implementing the logic. `Part a Logic` subsystem block contains the core logic for emergency detection. The rules to emergency detection are -

- if ( Acceleration < 80 && Audio Level < 28000 ) then emergecy level 1.
- if ( Acceleration >= 80 && Audio Level < 28000 ) then emergecy level 2.
- if ( Acceleration < 80 && Audio Level >= 28000 ) then emergecy level 2.
- if ( Acceleration > 80 && Audio Level > 28000 ) then emergecy level 3 (Highest level i.e. send GPS location).


Input / Output 

|  | Name | Type | Range |
| :---: | :---: | :---: | :---: |
| Input | Audio Amplitude | Continuous | 0 to inf  |
| Input | Acceleration | Continuous | 0 to inf |
| Output | Emergency Level | Discrete | 1, 2, 3 (3 is High)  |

### (Part b) Using Mamdani Fuzzy Logic

#### Files

- [`Part_b.slx`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/Part_b.slx)
- [`Receiver.slx`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/Receiver.slx)
- [`FuzzyLogic.fis`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/FuzzyLogic.fis)

#### Core Logic

This is the only change in this part. Rest other parts are same. `Part b Logic` subsystem block implements Mamdani Fuzzy Logic. Check file [`FuzzyLogic.fis`](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/FuzzyLogic.fis) for Fuzzy logic rules.

**Surface plot of input and output obtained from Mamdani Fuzzy Logic**

![Surface Plot](https://github.com/nishikantparmariam/Mamdani-Fuzzy-Logic-App/blob/3029b4561bb4f461410f1084beb35154e0830258/Fuzzy%20Logic%20Surface.png?raw=true)

Input / Output 

|  | Name | Type | Range |
| :---: | :---: | :---: | :---: |
| Input | Audio Amplitude | Continuous | 0 to inf  |
| Input | Acceleration | Continuous | 0 to inf |
| Output | Emergency Level | Continuous | 0 to 1 ( More that 0.8 is High)  |

## Contributors

- Nishikant Parmar (nishikant.parmar@iitgn.ac.in)
- Thakar Devanshu Nilesh (nilesh.thakar@iitgn.ac.in)

This was our submission for Assignment 6 of the course Nature Inpired Computing offered at IIT Gandhinagar in Academic Year 2021-22 in Semester-I under the guidance of **Prof. Nithin V. George (nithin@iitgn.ac.in)** and teaching assistant **Shekhar Kumar Yadav (yadav_shekhar@iitgn.ac.in)**.

## References

- Problem statement of Assignment 6 of the course ES 615: Nature Inspired Computing.
- https://in.mathworks.com/products/simulink.html
- https://in.mathworks.com/hardware-support/android-programming-simulink.html
- https://in.mathworks.com/help/supportpkg/android/ug/install-support-for-android-devices.html
- https://in.mathworks.com/help/supportpkg/android/examples.html?category=model-preparation&s_tid=CRUX_topnav


## Future Improvements

- Sending device's location using TCP Protocol if devices are far apart. See [here](https://in.mathworks.com/help/supportpkg/freescalefrdmk64fboard/ref/tcpsend.html?searchHighlight=tcp%20send&s_tid=srchtitle_tcp%20send_1).
- Designing a robust method to detect emergency, may be by using more inputs - temperature, altitude, battery percentage etc.

Feel free to fork, contribute and contact us.
