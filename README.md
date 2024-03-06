``` ____ ___ ____                    _             _ _           
 |  _ \_ _|  _ \    ___ ___  _ __ | |_ _ __ ___ | | | ___ _ __ 
 | |_) | || | | |  / __/ _ \| '_ \| __| '__/ _ \| | |/ _ \ '__|
 |  __/| || |_| | | (_| (_) | | | | |_| | | (_) | | |  __/ |   
 |_|_ |___|____/   \___\___/|_| |_|\__|_|  \___/|_|_|\___|_|   
  / _| ___  _ __  / ___|(_) ___ _ __ ___   ___ _ __  ___       
 | |_ / _ \| '__| \___ \| |/ _ \ '_ ` _ \ / _ \ '_ \/ __|      
 |  _| (_) | |     ___) | |  __/ | | | | |  __/ | | \__ \      
 |_|__\___/|_|    |____/|_|\___|_| |_|_|_|\___|_| |_|___/      
 |_   _(_) __ _      |  _ \ ___  _ __| |_ __ _| |              
   | | | |/ _` |_____| |_) / _ \| '__| __/ _` | |              
   | | | | (_| |_____|  __/ (_) | |  | || (_| | |              
   |_| |_|\__,_|     |_|   \___/|_|   \__\__,_|_|              
```                                                   

## Problem
I have always been deeply involved in control technology within the TIA Portal environment and have consistently found Siemens modules to be unsatisfactory. I require PI controllers for pressure regulation and PID controllers for temperature control. Here are a few issues that have bothered me:

- Different devices among different CPUs use different libraries.
- Simulation is not supported.
- Source code cannot be accessed or modified.
- Complexity is too high, requiring significant familiarization.
- Lack of portability to Codesys.
- Excessive integration within TIA Portal.
- Significant trial and error required until achieving functionality.
- Cyclic OBs and Global DBs are absolutely necessary. Integration within private functions is not possible.
- Manuals span thousands of pages.

## Work
After an extensive search for alternatives, I failed to find a suitable solution. Drawing inspiration from OSCAT, which is straightforward, I have rewritten functions for TIA14 and 15 compatible with 1200 and 1500 series CPUs. I continue to implement these controls in my TIA Portal 18 projects. My intention was not to cater to everyone but to contribute back to the community. Open libraries like OSCAT have provided me with valuable knowledge, and it is only fair that I share my work.
Please refrain from debating the functionality; these functions work as intended. Feel free to copy and use them. If you wish, share your experiences with my software with others.

## Usage Instructions
The controller produces outputs ranging from 0 to 100. When used with a binary actuator, utilize the clock generator for pulse width modulation. The PI controller is designed to operate independently, suitable for pressure regulation. The PID controller combines PI and D control, suitable for temperature regulation. Always stop the controller with the reset input if the regulation loop is disrupted. This prevents integral windup.

- ir_Input: The measured value of pressure or temperature.
- ir_Setpoint: The desired value of pressure or temperature.
- ir_ProportionalGain: The difference between ir_Input and ir_Setpoint is multiplied by this factor and added to the output value. Higher values result in a higher control response.
- ir_IntegrationGain: A triangular fuction is integrated. The integral value increases or decreases by the difference between ir_Input and ir_Setpoint per second. A high value results in rapid rise or fall of the integral value.
- ir_DerviateGain: The difference between ir_Input and ir_Setpoint is multiplied by the differential gain. Depending on the action time, the differential response decreases rapidly. A high value results in a high differential response.
- ir_DerviateActionTime: The duration in seconds until the differential response has halved. A high value results in a long differential response.
- ib_Reset: Empty the integral and set the output to zero.
- or_Output: Output value in % from 0 to 100.

![](PID-Control2.png)

## Installing
Installation is straightforward. Place the two SLC files under "External source files" and execute the menu item "Generate blocks from source."

![](Generate-blocks.png)

## Porting
Porting code is simple:
```
// Proportional
#Controller_Response_Proportional := #ir_ProportionalGain * (#ir_Setpoint - #ir_Input);
// Intergal
#Controller_Response_Integral += #ir_IntegrationGain * (#ir_Setpoint - #ir_Input) * #PastTime;
// Derviate
#Intermediate_value += (#ir_SetpointDiverence - #Intermediate_value) * #PastTime / #ir_DerivateActionTime;
#or_Output := (#ir_SetpointDiverence - #Intermediate_value) * #ir_DerivateGain;
```
Rest of code is mostly to prevent integral windup and ensure the cycle time is valid.

Note on porting to Step-7, Codesys, or similar:
```
#PastTime := LREAL_TO_REAL(RUNTIME(#StaticCycleTime_Aux));
IF #PastTime > 0 AND #PastTime < 0.1 THEN
```    
The first line returns the time between two calls in seconds with nanosecond accuracy. The second line checks validity. With Step-7, pass time as a parameter; with Codesys, use TIME_TCK. On 300 PLC, it might be possible to work with "SFC64" (TIMETICK).

## License
This project is released under the WTFPL LICENSE.
<a href="http://www.wtfpl.net/"><img src="http://www.wtfpl.net/wp-content/uploads/2012/12/wtfpl-badge-4.png" width="80" height="15" alt="WTFPL" /></a>
