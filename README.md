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
![](day4/day4sconscript.png)
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

