Minimal configuration of a core-v-mcu

## Prerequisite

1. [optional] If you wish to install the Conda enviroment with python 3.9:

```bash
$ conda update conda
$ conda env create -f environment.yml
```

Activate the environment with

```bash
$ conda activate core-v-mini-mcu
```
2. Install the required Python tools:

```
$ pip3 install --user -r python-requirements.txt
```

Add '--root user_builds' to set your build foders for the pip packages
and add that folder to the `PATH` variable

3. Install the required apt tools:

```
$ sudo apt install lcov libelf1 libelf-dev libftdi1-2 libftdi1-dev libncurses5 libssl-dev libudev-dev libusb-1.0-0 lsb-release texinfo makeinfo autoconf cmake flex bison libexpat-dev
```

In general, have a look at the [Install required software](https://docs.opentitan.org/doc/ug/install_instructions/#system-preparation) section of the OpenTitan documentation.

4. Install the RISC-V Compiler:

```
$ git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
$ cd riscv-gnu-toolchain
$ ./configure --prefix=/home/yourusername/tools/riscv --with-abi=ilp32 --with-arch=rv32imc --with-cmodel=medlow
$ make
```

Then, set the `RISCV` env variable as:

```
$ export RISCV=/home/yourusername/tools/riscv
```

5. Install the Verilator:

```
$ export VERILATOR_VERSION=4.210

$ git clone https://github.com/verilator/verilator.git
$ cd verilator
$ git checkout v$VERILATOR_VERSION

$ autoconf
$ ./configure --prefix=/home/yourusername/tools/verilator/$VERILATOR_VERSION
$ make
$ make install
```
Then, set the `PATH` env variable to as:

```
$ export PATH=/home/yourusername/tools/verilator/$VERILATOR_VERSION/bin:$PATH
```

In general, have a look at the [Install Verilator](https://docs.opentitan.org/doc/ug/install_instructions/#verilator) section of the OpenTitan documentation.

If you want to see the vcd waveforms generated by the Verilator simulation, install GTKWAVE:

```
$ sudo apt-get install -y gtkwave
```

## Adding external IPs

This repository relies on [vendor](https://docs.opentitan.org/doc/ug/vendor_hw/) to add new IPs.
In the ./util folder, the vendor.py scripts implements what is describeb above.

## Compiling for Verilator

```
$ fusesoc --cores-root . run --no-export --target=sim --tool=verilator --setup --build openhwgroup.org:systems:core-v-mini-mcu 2>&1 | tee buildsim.log
```

## Compiling for VCS

```
$ fusesoc --cores-root . run --no-export --target=sim --tool=vcs --setup --build openhwgroup.org:systems:core-v-mini-mcu 2>&1 | tee buildsim.log
```

## Compiling for Questasim

```
$ fusesoc --cores-root . run --no-export --target=sim --tool=modelsim --setup --build openhwgroup.org:systems:core-v-mini-mcu 2>&1 | tee buildsim.log
```
First set the env variable `MODEL_TECH` to your Questasim bin folder.

Questasim version must be >= Questasim 2019.3

You can also use vopt by running:

```
$ fusesoc --cores-root . run --no-export --target=sim_opt --setup --build openhwgroup.org:systems:core-v-mini-mcu 2>&1 | tee buildsim.log
```

## Compiling Software

Don't forget to set the `RISCV` env variable to the compiler folder (without the `/bin` included).

Then go to the `./sw` folder and type:

```
$ make applications/hello_world/hello_world.hex
```

This will create the executable file to be loaded in your target system (ASIC, FPGA, Questasim, Verilator, etc).

## Running Software on Verilator

Go to your target system built folder, e.g.

```
$ cd ./build/openhwgroup.org_systems_core-v-mini-mcu_0/sim-verilator
```

Then type:

```
$ ./Vtestharness +firmware=../../../sw/applications/hello_world/hello_world.hex
```

## Running Software on VCS

Go to your target system built folder, e.g.

```
$ cd ./build/openhwgroup.org_systems_core-v-mini-mcu_0/sim-vcs
```

Then type:

```
./openhwgroup.org_systems_core-v-mini-mcu_0 +firmware=../../../sw/applications/hello_world/hello_world.hex
```


## Running Software on Questasim

Go to your target system built folder, e.g.

```
$ cd ./build/openhwgroup.org_systems_core-v-mini-mcu_0/sim-modelsim/
```

or

```
$ cd ./build/openhwgroup.org_systems_core-v-mini-mcu_0/sim_opt-modelsim/
```

for the vopt version target.

Then type:

```
$ make run PLUSARGS="c firmware=../../../sw/applications/hello_world/hello_world.hex"
```

## FPGA Xilinx Nexys-A7 100T Flow

Work In Progress and untested!!!

To build and program the bitstream for your FPGA with vivado, type:

```
$ fusesoc --cores-root . run --no-export --target=nexys-a7-100t --setup --build openhwgroup.org:systems:core-v-mini-mcu 2>&1 | tee buildvivado.log
```

If you only need the synthesis implementation:

```
$ fusesoc --cores-root . build --no-export --target=nexys-a7-100t --setup openhwgroup.org:systems:core-v-mini-mcu 2>&1 | tee buildvivado.log
```

then

```
$ cd ./build/openhwgroup.org_systems_core-v-mini-mcu_0/nexys-a7-100t-vivado/
$ make synth
```

at the end of the synthesis, you can export your netlist typing:

```
$ vivado -notrace -mode batch -source ../../../hw/fpga/scripts/export_verilog_netlist.tcl
```

Only Vivado 2019.1.1 has been tried.

## UART DPI

We use the LowRISC OpenTitan [UART DPI](https://github.com/lowRISC/opentitan/tree/master/hw/dv/dpi/uartdpi) to simulated the UART peripheral. Read how to interact with it in the Section "Interact with the simulated UART" [here](https://docs.opentitan.org/doc/ug/getting_started_verilator/).
The output of the UART DPI module is printed in the `uart0.log` file in the simulation folder.

For example, to see the "hello world!" output of the Verilator simulation:

```
$ cd ./build/openhwgroup.org_systems_core-v-mini-mcu_0/sim-verilator
$ ./Vtestharness +firmware=../../../sw/applications/hello_world/hello_world.hex
$ cat uart0.log
```

## Change Peripheral Memory Map

```
$ python ./util/mcu_gen.py --cfg mcu_cfg.hjson --outdir ./hw/core-v-mini-mcu/include/ --pkg-sv ./hw/core-v-mini-mcu/include/core_v_mini_mcu_pkg.sv.tpl
```
```
python ./util/mcu_gen.py --cfg mcu_cfg.hjson --outdir ./sw/device/lib/runtime/ --header-c ./sw/device/lib/runtime/core_v_mini_mcu.h.tpl
```

## Files are formatted with Verible

We use version v0.0-1824-ga3b5bedf

See: [Install Verible](https://docs.opentitan.org/doc/ug/install_instructions/)
