# comp_archi_SoC
Computer Architechture SoC
### Day1
![](comparch/comparchi.png)
![](comparch/comparchi_simplepy.png)

### Day2
![](comparch/comparchi_two_level.py.png)
![](comparch/comparchi_two_levelm5out.png)
![](comparch/comparchi_two_levelm5outini.png)
![](comparch/comparchi_two_leveldebug.png)
![](comparch/comparchi_two_leveldebugRAM.png)
![](comparch/comparchi_two_leveldebugExec.png)
![](comparch/comparchi_two_levelXconscript.png)
![](comparch/comparchi_two_levelXconscript2.png)
![](comparch/comparchi_processcc.png)
![](comparch/comparchi_gem5optupdate.png)
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
![](assignment/gem5assignmentNewObject.png)
![](assignment/gem5assignmentSConscript.png)
![](assignment/gem5assignmentprocess.png)
![](assignment/gem5assignmentNewObjectupdated.png)
![](assignment/gem5assignmentRebuild.png)
![](assignment/gem5assignmentCreatingObject.png)
![](assignment/gem5assignmentCreatingObjectdebugMATRIX.png)
![](assignment/gem5assignmentCreatingObjectdebugRESULT.png)

