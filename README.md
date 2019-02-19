# E/gamma Energy Regression Trainer


This is the E/gamma Energy Regression Trainer used for RunII

It is https://github.com/cms-egamma/RegressionTrainer & https://github.com/cms-egamma/HiggsAnalysis ported to a common build system. It links against CMSSW but is otherwise standalone and is not build as part of CMSSW.

It is very much a legacy tool and E/gamma wishes to move away from it as quickly as possible. So you have been warned.

## setup instructions

First setup a CMSSW environment. We only link against this so we only need the CMSSW environment variables setup. Any version >=CMSSW_9_4_1 should work.

Then clone this repo into a location of your chosing. It does not have to be under $(CMSSW_BASE)/src, in fact it is better that it is not. 
```
git clone git@github.com:cms-egamma/EgRegresTrainerLegacy.git
cd EgRegresTrainerLegacy 
gmake RegressionTrainerExe -j 16
gmake RegressionApplierExe -j 16
./scripts/runSCRegJob.py  #edit scripts/runRegJob.py to your required parameters
```

This will run the training sequence for the sc regression workflow, which is fairly generic. It first generates the configs expected by RegressionTrainerExe in the config directory (or optionally elsewhere) and then does the training outputing the results in the results directory. It then runs the testing job where it takes the testing input and then obtains the correction and resolution estimate. It is intended to use the outputed tree as a friend to the input tree.

The training step will take all availible CPUs, ie if you have 24 cores, it'll automatically run 24 processes. The testing step runs over 4 threads which was emperically derived (this can be adjusted).

Then to make an example resolution plot:
```
root rootScripts/setupExample.c
hists = makeHists(regTestTree,{-3.0,-2.5,-2.,-1.6,-1.566,-1.4442,-1.1,-0.7,0.,0.7,1.1,1.4442,1.566,1.6,2.,2.5},150,0,1.5,{"sc.rawEnergy/mc.energy:sc.seedEta","sc.corrEnergy74X/mc.energy:sc.seedEta","regCorr.mean*sc.rawEnergy/mc.energy:sc.seedEta"},"mc.energy>0 && sc.sigmaIEtaIEta>0 && mc.dR<0.1 && mc.pt>20 && mc.pt<60");
compareRes({hists[0],"raw energy"},{hists[1],"current energy"},{hists[2],"new energy"},6); //6 is the bin number, adjust as you like
```


## build system 

This build system is a lightweight custom system Sam uses for his analysis code. Its ~11 years old and could have been better so has some quirks. 

### conventions
A libary is built for each subdirectoy of the "packages" directory. All c++ files must have the suffix ".cc". All header files must have the suffix of ".hh" although an individual package may allow them to be all ".h" instead. ROOT dictionaries are generated by a name_LinkDef.h in the dict dir

Files defining a main() function are located in the top level main directory. 

Include locations are #include "packagename/filename.hh"

### layout

main: location of all files which define a main() function. The filename is Name.cc. To build do "gmake NameExe -j 16" (where the -j 16 is just to build in parallel.

packages: location of the libary files. Each libary has a specific subdirectory here. 

packages/LibName\/include: location of header files

packages/LibName/src : location of src files

packages/LibName/dict : localtion of the LinkDef files

packages/LibName/package.mk: build fragment for the package, controlling what files are build as part of it

package.mk defines the LIBNAME_LIBFILES varible which controls the building of the libary. To add a file to the libary it should be added here as follows
LIBNAME_LIBFILES = $(PKG_OBJ_DIR)/FILENAME1.o $(PKG_OBJ_DIR)/FILENAME2.o 
where FILENAME1.cc and FILENAME2.cc exist in the src direction.

ROOT dictionaries are done by rootcint and are triggered by adding FILENAME1Dict.o to LIBNAME_LIBFILES. It requires that there is FILENAME1_LinkDef.h in the dict subdir.  Note here it matters on the header suffix, by default it assumes that it has FILENAME1.hh but it can be modified to FILENAME1.h on a per package basis by changing the rule 

LIBNAME/src/%_LinkDef.h: packages/LIBNAME/dict/%_LinkDef.h packages/LIBNAME/include/%.hh   to 

LIBNAME/src/%_LinkDef.h: packages/LIBNAME/dict/%_LinkDef.h packages/LIBNAME/include/%.h 

(ie deleting the last h)

Finally to make a new package just do coreScripts/mkPkg.py --pkgName NAME which will make the initial empty dirs and package.mk







