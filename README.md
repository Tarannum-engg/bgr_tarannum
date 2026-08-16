# BGR_sky130
This github repository is for the design of a Band Gap Reference Circuit (BGR) using Google-skywater130nm technology PDK.

## Introduction to BGR
The Bandgap Reference (BGR) is a circuit which provides a stable voltage output which is independent of factors like temperature, supply voltage. 

### Applications of BGR
- Low dropout regulators (LDO)
- DC-to-DC buck converters
- Analog-to-Digital Converter (ADC)
- Digital-to-Analog Converter (DAC)


## Contents
- [1. Tool and PDK Setup](#1-Tools-and-PDK-setup)
  - [1.1 Tools Setup](#1.1-Tools-setup)
  - [1.2 PDK Setup](#1.2-PDK-setup)
- [2. BGR introduction](#2-BGR-introduction)
  - [2.1 BGR Principle](#2.1-BGR-Principle)
  - [2.2 Types of BGR](#2.2-Types-of-BGR)
  - [2.3 Self-biased Current Mirror based BGR](#2.3-Self-biased-current-mirror-based-bgr)
- [3. Design and Prelayout Simulation](#3-Design-and-Prelayout-Simulation)
- [Layout Design](#Layout-design)
- [LVS and Post-layout Simulation](#LVS-and-post-layout-simulation)


## 1. Tools and PDK setup

### 1.1 Tools 
For the design and simulation of the BGR circuit we will need the following tools.
- Spice netlist simulation - [Ngspice]
- Layout Design and DRC - [Magic]
- LVS - [Netgen]

### 1.2 PDK setup

A process design kit (PDK) is a set of files used within the semiconductor industry to model a fabrication process for the design tools used to design an integrated circuit. The PDK is created by the foundry defining a certain technology variation for their processes. It is then passed to their customers to use in the design process.

The PDK we are going to use for this BGR is Google Skywater-130 (130 nm) PDK.

## 2. BGR Introduction

### 2.1 BGR Principle
A Bandgap Voltage Reference (BGR) works by adding two opposing voltages. One voltage goes down as temperature rises, and the other goes up. When added together, they cancel each other out. This creates a stable voltage of about 1.25V that does not change when the temperature changes.
<img width="749" height="455" alt="image" src="https://github.com/user-attachments/assets/8e89b95e-737a-4b0f-9cfa-5037ad5db526" />


#### 2.1.1 CTAT Voltage Generation

<img width="668" height="249" alt="image" src="https://github.com/user-attachments/assets/da7e3e19-6d8b-4e15-8b16-4128beead250" />


#### 2.1.2 PTAT Voltage Generation

<img width="804" height="405" alt="image" src="https://github.com/user-attachments/assets/2b0986a2-43df-4d12-8b0c-461333688199" />


### 2.2 Types of BGR
Architecture wise BGR can be designed in two ways

- Using Self-biased current mirror  
- Using Operational-amplifier 

Application wise BGR can be categorized as
- Low-voltage BGR
- Low-power BGR
- High-PSRR and low-noise BGR
- Curvature compensated BGR

### 2.3 Self-biased current mirror based BGR Circuit

The Self-biased current mirror based constitute of the following components.

- CTAT voltage generation circuit
- PTAT voltage generation circuit
- Self-biased current mirror circuit
- Reference branch circuit
- Start-up circuit

Advantages of SBCM BGR:

- Simplest topology
- Easy to design 
- Always stable

Limitations of SBCM BGR:

- Low power supply rejection ratio (PSRR)
- Cacode design needed to reduce PSRR
- Voltage head-room issue
- Need start-up circuit

## 3. Design and Pre-layout Simulation

### 3.1 Design Specification 
- Supply voltage = 1.8V
- Temperature: -40 to 125 Deg Cent.
- Power Consumption < 60uW
- Off current < 2uA
- Start-up time < 2us
- Tempco. Of Vref < 50 ppm


### 3.3 Circuit Design

**1. Current Calculation**

- Max. power Consumption < 60uW
- Max Total Current = 60 uW/1.8V=33.33uA
- So, we have chosen 10uA/branch, (3*10=30uA)
- Start-up current 1-2 uA

**2. Choosing Number of BJT in parallel in Branch2**
- Less number of BJT: require less resistance value but matching hampers
- More number of BJT: requires higher resistance value but gives good matching
- So a moderate number have chosen (8 BJT) for better layout matching and moderate resistance value.  

**3. Calculation of R1**
- R1= Vt* ln (8)/I =26 mv *ln(8)/10.7uA=5 KOhm
- R1 size: W=1.41um, L=7.8um, Unit res value: 2k Ohm
- Number of resistance needed: 2 in series and 2 in parallel (2+2+(2||2))

**4. Calculation of R2**
- Current through ref branch:I3=I2=Vt*ln(8)/R1
- Voltage across R2: R2*I3=R2/R1(Vt*ln(8))
- Slope of VR2= R2/R1 (ln(8)*115uv)/Deg Cent.
- Slope of VQ3=-1.6mV/Deg cent
- Adding both and equating to zero, R2 will be around 33k Ohm
- Number of resistance needed: 16 in series and 2 in parallel (2+2...+2+ (2||2))

**5. SBCM Design (Self-biased Current Mirror)**

***A. PMOS Design in SBCM***
- Make both the MP1 and MP2 well in Saturation 
- To reduce channel length modulation used L=2um
- Finally the size is **L=2u, W=5u and M=4**

***B. NMOS Design in SBCM***
- Make both the MN1 and MN2 either in Saturation or in deep sub-threshold
- We have made it in deep sub-threshold 
- To reduce channel length modulation used L=1um
- Finally the size is **L=1u, W=5u and M=8**

#### 3.3.1 Final Circuit
<img width="751" height="490" alt="image" src="https://github.com/user-attachments/assets/1b77d337-819e-4e62-b69c-c8c6fae8278c" />

### 3.4 Writing Spice netlist and Pre-layout simulation
As we are not using any schematic editor we have to write the spice netlist and simulate it using Ngspice.

#### 3.4.1 BGR using Ideal OpAmp

Now after simulating all our components, let's quick check our BGR behaviour using one VCVS as an ideal OpAmp. [netlist](/prelayout/bgr_using_ideal_opamp.sp)

In this simulation we should get the reference voltgae as an umbrella shaped curve and it should be ~1.2V.
<img width="739" height="560" alt="image" src="https://github.com/user-attachments/assets/09c7bcd6-e59d-4c79-8ea7-9dc5247ce65a" />

<img width="731" height="560" alt="image" src="https://github.com/user-attachments/assets/5691e799-7846-411f-aa21-e94931c0d024" />

<img width="738" height="572" alt="image" src="https://github.com/user-attachments/assets/90b29070-b719-4422-8ff0-ee8f87805fe6" />


#### 3.4.5 BGR with SBCM Pre Simulation

Now we will replace the ideal Op-Amp with self-biased current mirror which is our proposed design. We expect same type of output as in case of ideal OpAmp based BGR. We will also check for different corners, and will see how our circuit is performing in different corners.

<img width="844" height="618" alt="image" src="https://github.com/user-attachments/assets/a35f0b05-ebb1-4853-8a73-ec90e891cb8e" />

1. Vref vs temp
- Behaviour in TT corner [netlsit](/prelayout/bgr_lvt_rpolyh_3p40.sp)
<img width="742" height="567" alt="image" src="https://github.com/user-attachments/assets/57ae79ba-2492-467f-b27c-e22961429be3" />
Tempco. Of Vref = ~21.7 PPM

- Behaviour in SS corner
<img width="634" height="572" alt="image" src="https://github.com/user-attachments/assets/4a9007a2-a5b5-424b-9faa-067b25b56595" />

Tempco. Of Vref = ~43 PPM

- Behaviour in FF corner [netlist](/prelayout/bgr_lvt_rpolyh_3p40_ff.sp)
<img width="632" height="573" alt="image" src="https://github.com/user-attachments/assets/7fe7f7c8-6f23-4672-8085-15c773a54ce8" />

Tempco. Of Vref = ~10 PPM

- Behaviour in SS corner [netlist](/prelayout/bgr_lvt_rpolyh_3p40_ss.sp)
<p align="center">
  <img src="Images/prelayout/bgr_ss.png">
</p>

Tempco. Of Vref = ~45 PPM

2. Vref vs Vdd
   
<img width="634" height="569" alt="image" src="https://github.com/user-attachments/assets/d83403ac-6f0c-4714-9e45-73faf7189f23" />

3. transient 

<img width="620" height="557" alt="image" src="https://github.com/user-attachments/assets/ab6e8bf0-ba81-45a0-8586-1f27b1b0700b" />

#### 3.4.6 Extracted Netlist
<img width="835" height="781" alt="image" src="https://github.com/user-attachments/assets/b42158fe-c25e-4aba-b7bc-da103575698b" />





[Magic]:                http://opencircuitdesign.com/magic/
[Ngspice]:              http://ngspice.sourceforge.net
[Netgen]:               http://opencircuitdesign.com/netgen/
[NGSpiceMan]:           http://ngspice.sourceforge.net/docs/ngspice-html-manual/manual.xhtml
