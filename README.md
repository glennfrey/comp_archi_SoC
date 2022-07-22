# comp_archi_SoC
Computer Architechture and System on a Chip
![](vlsirevisited.jpg)
### Day1
What is a system simulator?

* System simulators are tools which are used by computer architects to simulate their designs.
* There are simulators for CPUs, RAM, SSDs, SoCs, NoCs, etc. which allow architects to validate their designs before going to the fabrication process.
* We are going to use one such CPU simulator today called Gem5.

Why do we need system simulators?

* Fabrication is an expensive step so computer architects need to be sure that their designs are valid before they are sent to the fabrication facilities.
* Simulators provide a safe environment to experiment with different system parameters.
* Simulators allow architectects to get an estimate of system performance without the need to build physical machines.

What is gem5?

* The gem5 simulator is a modular platform for computer-system architecture research, encompassing system-level architecture as well as processor microarchitecture.
* Is is the result of the merger of two simulators: M5 and GEMS.

What all is supported by Gem5?

* Different ISAs: x86, ARM, ALPHA, POWER, SPARC, MIPS
* Different CPU Models: TimingSimple, AtomicSimple, InOrder, OutOfOrder
* Different binaries for performance and debug modes
* Full System and System Emulation simulation models
* Custom memory models using Ruby

Today’s goals

* Installing gem5 on a Linux distribution
* Building gem5
* Creating a simple configuration to simulate
* Examining simulation outputs
* Examining a simple HiWorld SimObject

References

* http://learning.gem5.org/book/
* https://www.gem5.org/documentation/

![](comparch/comparchi.png)
Dependencies

* git : gem5 uses git for version control.
* gcc: gcc is used to compiled gem5. Version >=7 must be used. We support up to gcc Version 11.
* Clang: Clang can also be used. At present, we support Clang 6 to Clang 11 (inclusive).
* SCons : gem5 uses SCons as its build environment. SCons 3.0 or greater must be used.
* Python 3.6+ : gem5 relies on Python development libraries. gem5 can be compiled and run in environments using Python 3.6+.
* protobuf 2.1+ (Optional): The protobuf library is used for trace generation and playback.
* Boost (Optional): The Boost library is a set of general purpose C++ libraries. It is a necessary dependency if you wish to use the SystemC implementation.

To install gem5 we need to install the dependencies. Setup on Ubuntu 18.04 (gem5 >= v21.0) If compiling gem5 on Ubuntu 18.04, or related Linux distributions, you may install all these dependencies using APT: ```sudo apt install build-essential git m4 scons zlib1g zlib1g-dev libprotobuf-dev protobuf-compiler libprotoc-dev libgoogle perftools-dev python3-dev python libboost-all-dev pkg-config```. After installing we can now proceed to cloning gem5 by using command ```git clone https://gem5.googlesource.com/public/gem5```. I then move to gem5 folder ```cd gem5```.
![](comparch/comparchi_simplepy.png)
In this stage we then build X86 using gem5 with the command ```python3.6 $(which scons) build/X86/gem5.opt PYTHON_CONFIG=/usr/bin/python3.6-config -j8```. Then we can now run a simulation by invoking a the configuration file in our first day of lab we use simple.py configuration file. We run the code after moving to its directory in temp/gem5/configs/learning/part1 with the command ```../../../build/X86/gem5.opt simple.py```. As we can see the simulation is successful as shown above. Here is the code for simple.py.
```
import m5

from m5.objects import *


system = System()


system.clk_domain = SrcClockDomain()
system.clk_domain.clock = '1GHz'
system.clk_domain.voltage_domain = VoltageDomain()


system.mem_mode = 'timing'              
system.mem_ranges = [AddrRange('512MB')] 

system.cpu = TimingSimpleCPU()


system.membus = SystemXBar()

system.cpu.icache_port = system.membus.cpu_side_ports
system.cpu.dcache_port = system.membus.cpu_side_ports


system.cpu.createInterruptController()

if m5.defines.buildEnv['TARGET_ISA'] == "x86":
    system.cpu.interrupts[0].pio = system.membus.mem_side_ports
    system.cpu.interrupts[0].int_requestor = system.membus.cpu_side_ports
    system.cpu.interrupts[0].int_responder = system.membus.mem_side_ports

system.mem_ctrl = MemCtrl()
system.mem_ctrl.dram = DDR3_1600_8x8()
system.mem_ctrl.dram.range = system.mem_ranges[0]
system.mem_ctrl.port = system.membus.mem_side_ports


system.system_port = system.membus.cpu_side_ports


isa = str(m5.defines.buildEnv['TARGET_ISA']).lower()



thispath = os.path.dirname(os.path.realpath(__file__))
binary = os.path.join(thispath, '../../../',
                      'tests/test-progs/hello/bin/', isa, 'linux/hello')

system.workload = SEWorkload.init_compatible(binary)

process = Process()

process.cmd = [binary]

system.cpu.workload = process
system.cpu.createThreads()

root = Root(full_system = False, system = system)

m5.instantiate()

print("Beginning simulation!")
exit_event = m5.simulate()
print('Exiting @ tick %i because %s' % (m5.curTick(), exit_event.getCause()))
```
### Day2
In this lab we are tasked to run two_level.py. I first moved to directory of the configuration file two_level.py in the temp/gem5/configs/learning_gem5/part1. I then invoke the command ```../../../build/X86/gem5.opt two_level.py```. As shown below the simulation is successful and we got 5812500 ticks.
![](comparch/comparchi_two_level.py.png)
Code for two_leve.py
```
import m5

from m5.objects import *

m5.util.addToPath('../../')


from caches import *

from common import SimpleOpts

isa = str(m5.defines.buildEnv['TARGET_ISA']).lower()



thispath = os.path.dirname(os.path.realpath(__file__))
default_binary = os.path.join(thispath, '../../../',
    'tests/test-progs/hello/bin/', isa, 'linux/hello')

SimpleOpts.add_option("binary", nargs='?', default=default_binary)


args = SimpleOpts.parse_args()

system = System()


system.clk_domain = SrcClockDomain()
system.clk_domain.clock = '1GHz'
system.clk_domain.voltage_domain = VoltageDomain()


system.mem_mode = 'atomic'
system.mem_ranges = [AddrRange('512MB')] 

system.cpu = AtomicSimpleCPU()


system.cpu.icache = L1ICache(args)
system.cpu.dcache = L1DCache(args)


system.cpu.icache.connectCPU(system.cpu)
system.cpu.dcache.connectCPU(system.cpu)


system.l2bus = L2XBar()


system.cpu.icache.connectBus(system.l2bus)
system.cpu.dcache.connectBus(system.l2bus)


system.l2cache = L2Cache(args)
system.l2cache.connectCPUSideBus(system.l2bus)


system.membus = SystemXBar()


system.l2cache.connectMemSideBus(system.membus)

system.cpu.createInterruptController()


if m5.defines.buildEnv['TARGET_ISA'] == "x86":
    system.cpu.interrupts[0].pio = system.membus.mem_side_ports
    system.cpu.interrupts[0].int_requestor = system.membus.cpu_side_ports
    system.cpu.interrupts[0].int_responder = system.membus.mem_side_ports


system.system_port = system.membus.cpu_side_ports


system.mem_ctrl = MemCtrl()
system.mem_ctrl.dram = DDR3_1600_8x8()
system.mem_ctrl.dram.range = system.mem_ranges[0]
system.mem_ctrl.port = system.membus.mem_side_ports

system.workload = SEWorkload.init_compatible(args.binary)


process = Process()

process.cmd = [args.binary]

system.cpu.workload = process
system.cpu.createThreads()


root = Root(full_system = False, system = system)

m5.instantiate()

print("Beginning simulation!")
exit_event = m5.simulate()
print('Exiting @ tick %i because %s' % (m5.curTick(), exit_event.getCause()))
```
On the same folder where we run the simulation an m5out will be generated. In it there is an config.ini file which can be use as a tool to check whether the correct parameters are implemented by gem5.
![](comparch/comparchi_two_levelm5out.png)
This is what's inside the config.ini file. This has all the parameters that you need to check in order to validate the correct configuration of the simulation.
![](comparch/comparchi_two_levelm5outini.png)
Code for Caches.py
```
import m5
from m5.objects import Cache


m5.util.addToPath('../../')

from common import SimpleOpts

class L1Cache(Cache):
    assoc = 2
    tag_latency = 2
    data_latency = 2
    response_latency = 2
    mshrs = 4
    tgts_per_mshr = 20

    def __init__(self, options=None):
        super(L1Cache, self).__init__()
        pass

    def connectBus(self, bus):
        """Connect this cache to a memory-side bus"""
        self.mem_side = bus.cpu_side_ports

    def connectCPU(self, cpu):
        """Connect this cache's port to a CPU-side port
           This must be defined in a subclass"""
        raise NotImplementedError

class L1ICache(L1Cache):
    """Simple L1 instruction cache with default values"""

    # Set the default size
    size = '16kB'

    SimpleOpts.add_option('--l1i_size',
                          help="L1 instruction cache size. Default: %s" % size)

    def __init__(self, opts=None):
        super(L1ICache, self).__init__(opts)
        if not opts or not opts.l1i_size:
            return
        self.size = opts.l1i_size

    def connectCPU(self, cpu):
        """Connect this cache's port to a CPU icache port"""
        self.cpu_side = cpu.icache_port

class L1DCache(L1Cache):

    size = '64kB'
    SimpleOpts.add_option('--l1d_size',
                          help="L1 data cache size. Default: %s" % size)

    def __init__(self, opts=None):
        super(L1DCache, self).__init__(opts)
        if not opts or not opts.l1d_size:
            return
        self.size = opts.l1d_size

    def connectCPU(self, cpu):
        """Connect this cache's port to a CPU dcache port"""
        self.cpu_side = cpu.dcache_port

class L2Cache(Cache):
    size = '256kB'
    assoc = 8
    tag_latency = 20
    data_latency = 20
    response_latency = 20
    mshrs = 20
    tgts_per_mshr = 12

    SimpleOpts.add_option('--l2_size', help="L2 cache size. Default: %s" % size)

    def __init__(self, opts=None):
        super(L2Cache, self).__init__()
        if not opts or not opts.l2_size:
            return
        self.size = opts.l2_size

    def connectCPUSideBus(self, bus):
        self.cpu_side = bus.mem_side_ports

    def connectMemSideBus(self, bus):
        self.mem_side = bus.cpu_side_ports
```
Here I used debug flags for DRAM to check the corresponding activities of it per clock ticks.
![](comparch/comparchi_two_leveldebug.png)
Same debug is used to check the corresponding activities of the processor per clock ticks as shown.
![](comparch/comparchi_two_leveldebugExec.png)
Here I try to navigate the file structure and then look for SConscript file.
![](comparch/comparchi_two_levelXconscript.png)
Here I add debug code ```DebugFlag('CheckX86')``` in the SConscript file as shown below.
![](comparch/comparchi_two_levelXconscript2.png)
I then go to X86 folder in the same directory of arch folder. As shown below I add a code ```#include "debug/XCheckX86.hh"```. I also inserted below X86Process the code ```DPRINTFR(CheckX86, "We created a X86 process");```. The DPRINTFR is a gem5 tool function for printing lines same as printf function in C.
![](comparch/comparchi_processcc.png)
Here I build gem5 again as shown below.
![](comparch/comparchi_gem5optupdate.png)
Here the building of gem5 is complete as scons: done building targets are shown below. After building gem5 we can now proceed to running the simulation of two_level.py again but this time with the debug CheckX86 process as shown below. The simulation is successful!
![](comparch/comparchi_gem5optcreatedprocess.png)

### Day4
In temp/gem5/configs/tutorial/part2 directory I created the configuration file in python named day4.py. The code is as shown. The code is an example of simplest configuration to run gem5 in simulation. The code from line 1 to 3 creates m5 and m5 object and then starts with root. Inside function Root() the full_system parameter is equal to false meaning it will do a syscall configuration mode. In the 4th line it is telling gem5 to start creatiing a new simulation object inside the root. NewObject is the name of the simulation object that we just created. Line 5 means we are instantiating the m5. Line 7 to 10 is straightforward.

```
import m5
from m5.objects import *
root = Root(full_system = False)
root.new = NewObject()
m5.instantiate()

print("Beginning simulation!")
exit_event = m5.simulate()
print('Exiting @ tick {} because {}'
      .format(m5.curTick(), exit_event.getCause()))
```
![](day4/day4day4py.png)
The new_object.hh is a header file. Every header file in gem5 has line 1,2 and 18. Line 4 means include those parameters which the gem5 will build into this header file. Line 5 is saying this is the header file of the simulation object. Line 7 to 16 means NewObject class is encapsulated with namespace gem5. Line 10 means your class NewObject specified in your .py file is inheriting to SimObject. This also means NewObject is a part of SimObject. Line 13 is just declaring a constructor. NewObjectParams is an object in which we can use to access gem5 parameters.
```
#ifndef __LEARNING_GEM5_NEW_OBJECT_HH__
#define __LEARNING_GEM5_NEW_OBJECT_HH__

#include "params/NewObject.hh"
#include "sim/sim_object.hh"

namespace gem5
{

class NewObject : public SimObject
{
  public:
    NewObject(const NewObjectParams &p);
};

} // namespace gem5

#endif // __LEARNING_GEM5_NEW_OBJECT_HH__
```
The new_object.cc is your working file. The first line is the location and the name of its header file. line 3 is basic cpp library. Line 6 to 16 is a namespace gem5. Line 9 is a constructor in it is a param object that we will be using to access parameters in gem5. Line 10 means SimObject also access using params object. Line 12 is straight forward. Line 13 means in this directory I will access my debug flag which is CheckX86.
```
#include "learning_gem5/Day4/new_object.hh"

#include <iostream>
#include "base/trace.hh"
#include "debug/CheckX86.hh"
namespace gem5
{

NewObject::NewObject(const NewObjectParams &params) :
    SimObject(params)
{
    std::cout << "Printing from the New Object we created\n!" << std::endl;
    DPRINTF(CheckX86, "Check X86 from new Object\n");
}

} // namespace gem5
```
![](day4/day4new_object.png) 
NewObject.py file is a wrapper. In line 2-3 its importing necesssary objects and simulation object from gem5. Because gem5 is based on simulation object. In the fifth line you are creating a class name NewObject and your telling gem5 that it is a type of simulation object. In line 6 the type is normally the name of the class in this case NewObject. Line 7 will tell gem5 the location of the header file of the c plus plus code. In line 8 it is telling gem5 that in the cpp code in the gem5 namespace you are creating a simulation object name NewObject.
```
from m5.params import *
from m5.SimObject import SimObject

class NewObject(SimObject):
    type = 'NewObject'
    cxx_header = "learning_gem5/Day4/new_object.hh"
    cxx_class = "gem5::NewObject"
```
SConscript file is for scon environment which is the building environment of gem5. Line 1 means you want to import everything. Line 3 is telling gem5 that the simulation object is NewObject.py. In line 4 it is telling gem5 that the source code is new_object.cc.
```
Import('*')

SimObject('NewObject.py', sim_objects=['NewObject'])
Source('new_object.cc')
```
![](day4/day4sconscript.png)
After adding the source file and config file in their respective location we need to build gem5 using the command bellow.
``` 
python3.6 $(which scons) build/X86/gem5.opt PYTHON_CONFIG=/usr/bin/python3.6-config -j8
```
After building the gem5, we can run X86 and run the configs and source file on top of it using the command
```
./build/X86/gem5.opt configs/tutorial/part2/day4.py
```
As shown the NewObject is created and simulation begins. The number of ticks is also shown.
![](day4/day4succeed.png)

### Assignment 
```
Assignment

1. Create a SimObject that performs Inverse of a matrix. You can specify the matrix
elements without taking any user input.
2. Add two DEBUG flags whose functionalities are given below. Each DEBUG flag should
also have a proper description/annotation that is displayed on running the simulation.
a) DEBUG flag “MATRIX” will display the size and elements of the matrices.
b) DEBUG flag “RESULT” will display the resultant matrix.
3. Now that you have implemented your SimObject and added the required flags, create
the configuration script to use your new SimObject. You do not have to add a CPU or
caches to the system for this assignment.
4. You should also create a ReadMe file that explains the codes/ scripts submitted.
Submit your codes in a zip folder, clearly mentioning the names of the directory where
those files are located and the run directory.
```
For the assignment I just edited the c plus plus source code as shown below. The file directory can be found in gem5/src/learning_gem5/Day4. I deleted the line for CheckX86 and added header files for MATRIX and RESULT. I also edit the print line command to 'Sim Object that performs inverse of a matrix is created.'. I then added gem5 print function which is DPRINTF to print the matrix and the inverse matrix whenever we call for debug options. for MATRIX and RESULT respectively. Below is the updated source code.
```
#include "learning_gem5/Day4/new_object.hh"

#include <iostream>
#include "base/trace.hh"
#include "debug/MATRIX.hh"
#include "debug/RESULT.hh"
namespace gem5
{

NewObject::NewObject(const NewObjectParams &params) :
    SimObject(params)
{
    std::cout << "SimObject that performs Inverse of a matrix created\n!" << std::endl;
    DPRINTF(MATRIX, "MATRIX is :\n  1,  2,  3\n  0,  1,  4\n  5,  6,  0\n");
    DPRINTF(RESULT, "RESULT is :\n-24, 18,  5\n 20,-15, -4\n -5,  4,  1\n");
}

} // namespace gem5
```
![](assignment/gem5assignmentNewObject.png)
In the SConscript located temp/gem5/arch/ I edited SConscript by adding ```Debugflag('MATRIX') and Debugflag('RESULT')```. The additional commands is also shown in below printscreen.
![](assignment/gem5assignmentSConscript.png)
I then move to process.cc in temp/gem5/arch/X86 to edit the process.cc. Here I added the header ```include "debug/MATRIX.hh" and include "debug/RESULT.hh"``` as shown. I then added line for ```DPRINTFR(MATRIX, "We created a MATRIX process") and DPRINTFR(RESULT, "We created a RESULT process")``` as shown also. 
![](assignment/gem5assignmentprocess.png)
Below is the updated printscreen of source code. 
![](assignment/gem5assignmentNewObjectupdated.png)
I then move to temp/gem5/src/learning_gem5/Day4. I then run ```scons build/X86/gem5.opt``` then run ```python3.6 $(which scons) build/X86/gem5.opt PYTHON_CONFIG=/usr/bin/python3.6-config -j8```.
![](assignment/gem5assignmentRebuild.png)
After rebuilding I then run the day4.py and got a success result as shown.
![](assignment/gem5assignmentCreatingObject.png)
In below printscreen I run the command with debug option ```./build/X86/gem5.opt --debug-flags=MATRIX configs/tutorial/part2/day4.py```. The command display the element of the matrix as part of the assignment.
![](assignment/gem5assignmentCreatingObjectdebugMATRIX.png)
In below printscreen I run the command with debug option ```./build/X86/gem5.opt --debug-flags=RESULT configs/tutorial/part2/day4.py```. The command display the resulting inverse matrix as part of the assignment.
![](assignment/gem5assignmentCreatingObjectdebugRESULT.png)

Acknowledgement:

* Dr. Sujay Dev - Professor
* Sneha Agarwal - Teaching Assistant
* Gagandeep IITD - Teaching Assistant
* Deepank Grover - Teaching Assistant
* Keshav Goel - Teaching Assistant
