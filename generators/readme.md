
## IP wrapper generator for AMBA APB and AHB Lite

``amba_wrap.py ip.yml|ip.json -apb|-ahbl -tb|-ch|-md``
- Options:
    - `-apb` : generates an APB wrapper.
    - `-ahbl` : generates an AHB Lite wrapper.
    - `-tb` : generates a Verilog testbench for the generated bus wrapper.
    - `-ch` : generates a C header file containing the register definitions.
    - `-md` : generates documentation in MD and Bitfield formats.
- Arguments:
    - `ip.yml|ip.json`: A YAML/JSON file that contains the IP definition.

## YAML Template Generator

``v2yaml.py IP.v module_name``

## A Typical Workflow

1. Describe the IP in YAML or JSON format. The format is outlined in the following section. To make things easier, ```v2yaml.py``` may be used to generate a template YAML file from the IP RTL Verilog file with some of the sections filled automatically for you.

2. [Optional] Convert the YAML file into JSON using tools such as [this one](https://onlineyamltools.com/convert-yaml-to-json).

3. Generate the wrapper RTL by invoking: 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
``amba_wrap.py ip.yml|ip.json -apb|-ahbl > ip_APB.v``

4. Genertate a template testbench.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
``amba_wrap.py ip.yml|ip.json -apb|-ahbl -tb > ip_APB_tb.v``

5. Generate the Register Definitions C header file needed for FW development.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
``amba_wrap.py ip.yml|ip.json -apb|-ahbl -ch > ip_APB.h``

6. Generate an, almost, complete Markdown documentation of the IP. This includes register/fields tables as well as graphics for the register fields in the bitfield format.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
``amba_wrap.py ip.yml|ip.json -apb|-ahbl -md > ip_APB.md``

## YAML IP Definition File Format

A YAML file is used to capture the IP information. This includes:

### General Information

Basic information about the IP like the author, the license, etc. For an example:
```yaml
info: 
  name: "MS_GPIO"
  description: "An 8-bit bi-directional General Purpose I/O (GPIO) with synchronizers and edge detectors."
  repo: "github.com/shala/MS_GPIO"
  owner: "AUCOHL"
  license: "MIT"
  author: "Mohamed Shalan"
  email: "mshalan@aucegypt.edu"
  version: "v1.0.0"
  date: "3-18-2022"
  category: "digital"
  tags: 
    - peripheral
    - GPIO
  bus: 
    - generic
  type": "soft"
  status: "verified"
  cell_count: "690"
  width": "0.0"
  height": "0.0"
  technology: "n/a"
  clock_freq_mhz: "10"
  digital_supply_voltage: "n/a"
  analog_supply_voltage: "n/a"
  ```

### Parameter Definitions

This section is used if the IP RTL model is parameterized. The parameters defined in this section can be used in other sections to specify the width of fields and registers.

```YAML
parameters:
  - name: SC
    default: 8
  - name: MDW
    default: 9
  - name: GFLEN
    default: 8
  - name: FAW
    default: 4
```
### Port Definitions

IP Port definitions. For an example:
```yaml
ports:
  - name: "data_in"
    width: 8
    direction: input
    description: "The input data"
  - name: "data_out"
    width: 8
    direction: output
    description: "The output data"
```
### External Interface Definitions
IP External Interfaces to other sub-systems. For an example:
```yaml
external_interface: 
  - name: "i_pad_in"
    port: "pad_in"
    direction: "input"
    width: 8
    description: The input pads
  - name: "o_pad_out"
    port: "pad_out"
    direction: "output"
    width: 8
    description: The output pads
```

### Clock and Reset Definitions

```YAML
clock:
  name: clk

reset:
  name: rst_n
  level: 0
```

### Register Definitions

Register definitions. For an example:
```yaml
registers:
- name: CAP
  size: 16
  mode: r
  fifo: no
  offset: 8
  bit_access: no
  read_port: capture
  description: The captured value.
- name: CTRL
  size: 4
  mode: w
  fifo: no
  offset: 12
  bit_access: no
  description: Control Register.
  fields:
  - name: TE
    bit_offset: 0
    bit_width: 1
    write_port: tmr_en
    description: Timer enable
  - name: CE
    bit_offset: 1
    bit_width: 1
    write_port: cntr_en
    description: Counter enable
```
- The ``mode`` property can be set to: 
  - ``w`` for registers that are meant for writing only; reading from it returns the last written data value.
  - ``r`` for registers that are meant for reading only; hence they cannot be written. 
  - ``rw`` for registers that are read and written differently; for example, the data register of a GPIO peripheral. Reading this register returns the data provided on input GPIO pins and writing the register sets the values of output GPIO pins.

- The ``bit_access`` property is used to enable bit-level access (Not implemented functionality).
- The ``fifo`` property is used to specify whether this register is used to access a FIFO. If it is set to ``yes`` the FIFO has to be defined.

### FIFO Definitions

```yaml
fifos:
  - type: read  
    width: MDW
    depth: 16 
    register: rxdata
    data_port: rdata
    control_port: rd
  - type: write
    width: MDW
    depth: 16
    register: txdata
    data_port: wdata
    control_port: wr
```


### Event Flag Definitions

Event flags are used for generating interrupts. For an example:

```yaml
flags:
- name: CM
  port: cntr_match
  description: Counter match.
- name: CAP
  port: cap_done
  description: Capture is done.
```

