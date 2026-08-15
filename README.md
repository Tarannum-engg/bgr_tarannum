# BGR_sky130
This github repository is for the design of a Band Gap Reference Circuit (BGR) using Google-skywater130nm technology PDK.

## Introduction to BGR
The Bandgap Reference (BGR) is a circuit which provides a stable voltage output which is independent of factors like temperature, supply voltage. 
<p align="center">
  <img src="Images/BGR1.png">
</p>

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
<p align="center">
  <img src="Images/BGR_Principle.png">
</p>

#### 2.1.1 CTAT Voltage Generation

<p align="center">
  <img src="Images/CTAT.png">
</p>

#### 2.1.2 PTAT Voltage Generation
<p align="center">
  <img src="Images/Equation.png">
</p>


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
<p align="center">
  <img src="Images/finalbgr.png">
</p>

### 3.4 Writing Spice netlist and Pre-layout simulation
As we are not using any schematic editor we have to write the spice netlist and simulate it using Ngspice.

**Steps to write a netlist**

#### 3.4.1 CTAT Simulation

**CTAT Voltage generation with single BJT** [netlist](/prelayout/ctat_voltage_gen.sp)

In this simulation we take a BJT as a diode, Provide a Current source of 10uA and we need to find the volatge variation across the BJT with respect to the temp.

After simulation we can get a wavefrom like below, and from the wavefrom we can see the CTAT behaviour of the BJT, and can find the slope.
<p align="center">
  <img src="Images/prelayout/ctat@2v.PNG">
</p>

**CTAT Voltage generation with Multiple BJT** [netlist](/prelayout/ctat_voltage_gen_mul_bjt.sp)

In this simulation we will check the CTAT voltage across the 8 parallel connected BJTs.
<p align="center">
  <img src="Images/prelayout/ctat_mul_bjt.png">
</p>

As we can see the slope is increasing in case of multiple BJTs.

**CTAT Voltage generation with different current source values** [netlist](prelayout/ctat_voltage_gen_var_current.sp)

In this simulation we will check the CTAT voltage dependancy on current.
<p align="center">
  <img src="Images/prelayout/ctat_cur.png">
</p>

#### 3.4.2 PTAT Simulation

**PTAT Voltage generation with ideal current source** [netlist](/prelayout/ptat_voltage_gen_ideal_current_source.sp)

In this simulation we will take one ideal current source and will connect it to 5K Ohm resistance and 8 parallel BJTs. From this we will find the voltgae difference between the two terminals of the resistnce, which will give us a slightly PTAT voltage.
<p align="center">
  <img src="Images/prelayout/ptat_cir.png">
</p>

We can find that the voltage V(ra1)-V(qp2) is increasing with temp. which is the desired PTAT voltage.

**PTAT Voltage generation with VCVS** [netist](/prelayout/ptat_voltage_gen.sp)

In this simulation we will check the amplified PTAT voltage using one VCVS.
<p align="center">
  <img src="Images/prelayout/ptat_vcvs.png">
</p>


#### 3.4.3 Resistance tempco.

We know that resistor also behaves as PTAT, i.e the voltage across the resistor also increases with increase in the temp. In our BGR the PTAT voltage we are getting is not only by the virtue of Vt(Thermal voltgae) but with the additional PTAT voltage of the resistance.

In this simulation we will check the tempco. of resistor using ideal current source of 10uA. [netlist](/prelayout/res_tempco.sp)
<p align="center">
  <img src="Images/prelayout/res_tempco_v.png">
</p>

From the above curve we can find that the Voltage across the resistnace is increasing with increase in temp., i.e. the PTAT nature.

Now to find the temco. we have to find the change in resistance w.r.t temp. The tempco. can be found from the slope of the following curve.
<p align="center">
  <img src="Images/prelayout/res_tempco.png">
</p>

Also we can find the PTAT voltages across the resistance for different current values from the following curve. [netlist](prelayout/res_tempco_var_current.sp)
<p align="center">
  <img src="Images/prelayout/res_tempco_var_i.png">
</p>

#### 3.4.4 BGR using Ideal OpAmp

Now after simulating all our components, let's quick check our BGR behaviour using one VCVS as an ideal OpAmp. [netlist](/prelayout/bgr_using_ideal_opamp.sp)

In this simulation we should get the reference voltgae as an umbrella shaped curve and it should be ~1.2V.
<p align="center">
  <img src="Images/prelayout/ideal_bgr.png">
</p>

#### 3.4.5 BGR with SBCM

Now we will replace the ideal Op-Amp with self-biased current mirror which is our proposed design. We expect same type of output as in case of ideal OpAmp based BGR. We will also check for different corners, and will see how our circuit is performing in different corners.

- Behaviour in TT corner [netlsit](/prelayout/bgr_lvt_rpolyh_3p40.sp)
<p align="center">
  <img src="Images/prelayout/bgr_tt.png">
</p>

Tempco. Of Vref = ~21.7 PPM

- Behaviour in FF corner [netlist](/prelayout/bgr_lvt_rpolyh_3p40_ff.sp)
<p align="center">
  <img src="Images/prelayout/bgr_ff.png">
</p>

Tempco. Of Vref = ~10 PPM

- Behaviour in SS corner [netlist](/prelayout/bgr_lvt_rpolyh_3p40_ss.sp)
<p align="center">
  <img src="Images/prelayout/bgr_ss.png">
</p>

Tempco. Of Vref = ~45 PPM





[Magic]:                http://opencircuitdesign.com/magic/
[Ngspice]:              http://ngspice.sourceforge.net
[Netgen]:               http://opencircuitdesign.com/netgen/
[NGSpiceMan]:           http://ngspice.sourceforge.net/docs/ngspice-html-manual/manual.xhtml
