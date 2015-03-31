## PID with Lego Mindstorm and NXC ##
### Line follower ###
**Documentation for line (edge) following program**: Check the **[source code](https://code.google.com/p/lego-mindstorm-examples/source/browse/followline/pidOndraPlatek.nxc)**

---

I can lend a Lego Mindstorm from brick [UNIBZ](http://www.unibz.it) and I discovered browsing internet NXC language, which does not require changing firmware (and backing up all other teachers and students programs)


Several steps followed:)
  1. Learn using Light Sensor and robot built exactly like in NXC
> > [tutorial](http://bricxcc.sourceforge.net/nbc/nxcdoc/NXC_tutorial.pdf)
> > and decided for reading raw format
  1. Tested printing on LCD screen
```
      // print message with offset 0 on 2nd line of LCD display
      TextOut(0, LCD_LINE2, "my string message");
      NumOut(0, LCD_LINE2, lineColor); // prints integer analogically 
```
  1. For parameters it's really useful to look at [Doxygen NXC help](http://bricxcc.sourceforge.net/nbc/nxcdoc/nxcapi/index.html)
  1. I implemented PID control systems. The main of line following with light sensor is to keep the robot on the edge of the color of line and the environment. It looks like two discrete values, but it turns out that is better to think about it like it is proportional. So now comes PID, which main idea is to smooth oscillations caused by balancing on the narrow edge so hard. The P stands for **Propotional**, which means we put the less speed to right motor the we think we are too much at left side and similar the other way around. **Derivation** should eliminate the oscillation and **Integration** should remind us if we are deviating even a little but for a long time that it is still not good approach. That is **PID** very **lame said**. Better reading can be found [here](http://www.inpharmix.com/jps/PID_Controller_For_Lego_Mindstorms_Robots.html).


---

#### Tested PID Configuration ####

> I use following "equation" to regulate the direction of robot
```
    turn = SCALE *((P * err ) + (I * errI) + (D * errD)) ; // scale is macro => CONSTANT
    left = SPEED + turn;
    right = SPEED - turn;
```

  * **P** stands for Proportional **coefficient**
```
 P = ((0.6*KC) + P_GUESS )
```
  * **I** stands for Integrative **coefficient**
```
 I = ((2*P*dT/PC) + I_GUESS)
```
  * **D** stands for Derivative **coefficient**
```
 D = ((0.125*P*PC/dT) + D_GUESS)
```

> #### Another values ####
  * I use the **Critical Point** value **KC** in order to compute P,I and D.
> The coeficient **KC** is empirically determined by setting I=D=0 and increasing P till the oscillation of robot, which follows the line do not allow the robot to follow the line any more!
  * **dT** how frequently I compute _turn_ and update motor speed in Seconds e.g. 0.02s
  * **Pc** Oscillation period (how long the it takes the robot to swing from one side to the other side). It's enough to use just rough estimation

| SPEED| KC |  PC |  dT  |  P  |   I |  D   |P\_GUESS|I\_GUESS|D\_GUESS| My Result |
|:-----|:---|:----|:-----|:----|:----|:-----|:-------|:-------|:-------|:----------|
| 20   | 50 | 0.5 | 0.02 | 30  | 2.4 | 94   |   0 | 0 | 0| oscillate a lot |
| 20   | 50 | 0.5 | 0.11 | 30  | 13 | 17   |   0 | 0 | 0| go backwards:).. I wanted other values |
| 20   | 50 | 0.5 | 0.02 | 30  | 1.4 | 44   |  0 | -1 | -50| Little bit better still huge oscillation |
| 20   | 50 | 0.5 | 0.02 | 25  | 0.5 | 8.13   |  -5 | -1.5 | -70| Slow but smooth with one side oscillation |
| 20   | 50 | 0.5 | 0.02 | 25  | 0.8  | 18.2   |  -5 | -1.2 | -60| Slow but even smoother|
| 20   | 50 | 0.5 | 0.02 | 30  | 1.2  | 33.8   |  0 | -1.2 | -60| Oscillate lot but smoothly|
| 20   | 50 | 0.5 | 0.02 | 30  | 1  | 43.8   |  0 | -1.4 | -50| so far best solution, at start has to placed on line!|
| 20   | 50 | 0.5 | 0.02 | 30  | 0.5  | 43.8   |  0 | -1.9 | -50| Even better!|
| 20   | 50 | 0.5 | 0.02 | 30  | 0.2  | 43.8   |  0 | -2.5 | -50| About the same|
| 20   | 50 | 0.5 | 0.02 | 30  | **-0.2**  | 53.8   |  0 | **-2.6** | -40| Loses the line|
| 20   | 50 | 0.5 | 0.02 | 30  | **0.1**  | 53.8   |  0 | **-2.35** | -40| **I am satisfied:)**  |