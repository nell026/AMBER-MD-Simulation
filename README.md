# AMBER-MD-Simulation

Setting up a DNA-Ligand System for AMBER/WESTPA Simulations
**parameters subject to change based on system studied**

AMBER

Find a PDB of your system.

Download the PDB of the DNA-Ligand into your directory.

Clean up the PDB.

Make a directory for your DNA, DNA-Ligand complex, and just the ligand. 

Ligand needs to go through antechamber 

Antechamber Procedure for Ligand 

1.	Ligand PDB -> GCRT (gaussian input)
2.	GCRT -> GESP
3.	GESP -> mol2/prepi file
4.	mol2/prepi -> frcmod file


pdb4amber -i orig.pdb -o new.pdb –reduce –dry

antechamber -i file.pdb -fi pdb -o file.com -fo gcrt -gv 1 -ge file.gesp -nc 1 (for net charge=1)

**run gaussian job** -> generate gesp file

antechamber -i file.gesp -fi gesp -o file.mol2 -fo mol2 -c resp -eq 2

antechamber -i file.gesp -fi gesp -o file.prepi -fo prepi -c resp -eq 2 

parmchk2 -i file.mol2 -f mol2 -o file.frcmod

**make off files now**

Creating off (.lib) files

module load amber/
tleap/
source leaprc.gaff2/
MOL=loadmol2 file.mol2/
check MOL/
loadamberparams file.frcmod/
saveoff MOL file.lib/
saveamberparm MOL file.prmtop file.rst7/
quit/

Once completed with the antechamber protocol, the following are the next steps:

###To create your system###

LIGAND ONLY PREP/

source leaprc.gaff2/
source leaprc.water.spce/
loadoff distamycin_B_H.off/
loadamberprep distamycin_B_H.prepi/
loadamberparams distamycin_B_H.frcmod/
MOL = loadpdb distamycin_B_H.pdb/
addions MOL Cl- 0/
solvateoct MOL SPCBOX 12.0/
check MOL/
savepdb MOL final_distamycinB.pdb/
saveamberparm MOL final_distamycinB.prmtop final_distamycinB.inpcrd/
saveamberparm MOL final_distamycinB.prmtop final_distamycinB.rst7/
quit/


LIGAND-DNA COMPLEX PREP 1:1/

source leaprc.DNA.bsc1/
source leaprc. gaff2/
source leaprc. water.spce/
loadoff distamycin_B_H.off/
loadamberprep distamycin_B_H.prepi/
loadamberparams distamycin_B_H.frcmod/
DMY = loadpdb dna_distamycin.pdb/
addions DMY Na+ 0/
solvateoct DMY SPCBOX 12.0/
check DMY/
savepdb DMY final_dna_distamycin.pdb/
saveamberparm DMY final_dna_distamycin.prmtop final_dna_distamycin.inpcrd/
saveamberparm DMY final_dna_distamycin.prmtop final_dna_distamycin.rst7/
quit/


LIGAND-DNA COMPLEX PREP 2:1/

source leaprc.DNA.bsc1/
source leaprc.gaff2/
source leaprc.water.spce/
loadoff distamycin_A_H.off/
loadamberprep distamycin_A_H.prepi/
loadamberparams distamycin_A_H.frcmod/
loadoff distamycin_B_H.off/
loadamberprep distamycin_B_H.prepi/
loadamberparams distamycin_B_H.frcmod/
DMY = loadpdb double_bound_final.pdb/
addions DMY Na+ 0/
solvateoct DMY SPCBOX 12.0/
check DMY/
savepdb DMY final_double_bound.pdb/
saveamberparm DMY final_double_bound.prmtop final_double_bound.inpcrd/
saveamberparm DMY final_double_bound.prmtop final_double_bound.rst7/
quit/


DNA ONLY PREP/

source leaprc.DNA.bsc1/
source leaprc.water.spce/
mol = loadpdb dna_only.pdb/
addions mol Na+ 0/
solvateoct mol SPCBOX 12.0/
savepdb mol dna_only_solv.pdb/
saveamberparm mol dna_only.prmtop dna_only.inpcrd/
saveamberparm mol dna_only.prmtop dna_only.rst7/
check mol /
quit/


Once you have created these files (your system), you may proceed with the “New biological molecule (DNA/INC) protocol":

1.	Initial minimization: Let solvent relax around restrained solute
-	1000 steps (500 steepest descents/500 conjugate gradient)
-	500kcal/mol-Å2 restraints (solute)
2.	Second minimization: Let everything relax 
-	2500 steps (1000 steepest descents/1500 conjugate gradient)
3.	Defrost (md1): Begin constant volume to warm to proper temperature with restrained solute
-	100ps NPT 
-	Langevin temperature control 0 -> 300K
-	25 kcal/mol-Å2 restraints (solute)
4.	Equilibration (md2): Switch to constant pressure to get proper density while gradually releasing restraints on solute (5-stage release with strong restraints for first 40ps while density is changing most rapidly)250ps NPT
-	Langevin temperature control 300K
-	“Weak-coupling” pressure control 1.0bar (~1atm)
-	25 -> 5kcal/mol-Å2 restraints(solute)
o	md2a 25kcal 50ps
o	md2b 20kcal 50ps
o	md2c 15kcal 50ps
o	md2d 10kcal 50ps
o	md2e 5kcal 50ps
5.	Equilibration (md3): Release solute restraints and collect data to isotropically scale box size to reflect average volume 
-	200ps NPT
-	Langevin temperature control 300K 
-	“Weak-coupling” pressure control 1.0bar(~1.0atm)
-	**calculate new volume and replace x,y,z in restart file**
-	New flag: ntxo=1 
6.	Equilibration (md4): Switch to constant volume and equilibrate with scaled box size 
-	1ns NVT
-	Remove ntxo=1 flag 
7.	Production run: keep the same conditions as the equilibration run
-	1 microsecond
-	Keep running production for however long is needed for your system until it equilibrates completely. 



Input Files for Protocol

Min1.in /

minimization for water and counterions/
 &cntrl/
  irest=0,/
  ntx=1,/
  imin=1,/
  ntr=1,/
  ncyc=500,/
  maxcyc=1000,/
  ntr=1,/
  restraint_wt=500.0,/
  restraintmask=':1',/ ##based on what you want to restrain###
  ntpr=100,/
  ntwx=0,/
  cut=10.0,/
 /


Min2.in /

Solute minimization run/
&cntrl/
  imin=1,/
  ntx=1,/
  irest=0,/
  ncyc=1000,/
  maxcyc=2500,/
  ntpr=100,/
  ntwx=0,/
  cut=10.0,/
 /


Md1.in/

Defrost NVT 0 to 300 K with res on DNA/
 &cntrl/
  imin=0,/
  irest=0,/
  ntx=1,/
  ntb=1,/
  cut=10.0,/
  ntr=1,/
  restraint_wt=25.0, restraintmask=':1',/
  ntc=2,/
  ntf=2,/
  tempi=0, temp0=300,/
  ntt=3, gamma_ln=1.0,/
  nstlim=50000, dt=0.002,/
  ntpr=100, ntwx=100, ntwr=1000/
 /


Md2a.in/

Minimization Restraint 25kcal 50ps/
 &cntrl/
  imin=0,/
  irest=1,/
  ntx=5,/
  ntb=2,/
  pres0=1.0,/
  ntp=1,/
  taup=1.0,/
  cut=10.0,/
  ntr=1, restraint_wt=25.0, restraintmask=':1',/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3,/
  gamma_ln=1.0,/
  nstlim=25000, dt=0.002,/
  ntpr=100, ntwx=1000, ntwr=1000/
 /


Md2b.in/

Minimization Restraint 20kcal 50ps/
 &cntrl/
  imin=0,/
  irest=1,/
  ntx=5,/
  ntb=2,/
  pres0=1.0,/
  ntp=1,/
  taup=1.0,/
  cut=10.0,/
  ntr=1, restraint_wt=20.0, restraintmask=':1',/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3,/
  gamma_ln=1.0,/
  nstlim=25000, dt=0.002,/
  ntpr=100, ntwx=1000, ntwr=1000/
 /


Md2c.in/

Minimization Restraint 15kcal 50ps/
 &cntrl/
  imin=0,/
  irest=1,/
  ntx=5,/
  ntb=2,/
  pres0=1.0,/
  ntp=1,/
  taup=1.0,/
  cut=10.0,/
  ntr=1, restraint_wt=15.0, restraintmask=':1',/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3,/
  gamma_ln=1.0,/
  nstlim=25000, dt=0.002,/
  ntpr=100, ntwx=1000, ntwr=1000/
 /


Md2d.in/

Minimization Restraint 10kcal 50ps/
 &cntrl/
  imin=0,/
  irest=1,/
  ntx=5,/
  ntb=2,/
  pres0=1.0,/
  ntp=1,/
  taup=1.0,/
  cut=10.0,/
  ntr=1, restraint_wt=10.0, restraintmask=':1',/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3,/
  gamma_ln=1.0,/
  nstlim=25000, dt=0.002,/
  ntpr=100, ntwx=1000, ntwr=1000/
 /


Md2e.in/

Minimization Restraint 5kcal 50ps/
 &cntrl/
  imin=0,/
  irest=1,/
  ntx=5,/
  ntb=2,/
  pres0=1.0,/
  ntp=1,/
  taup=1.0,/
  cut=10.0,/
  ntr=1, restraint_wt=5.0, restraintmask=':1',/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3,/
  gamma_ln=1.0,/
  nstlim=25000, dt=0.002,/
  ntpr=100, ntwx=1000, ntwr=1000/
 /


Md3.in/

Equilibrization md3/
 &cntrl/
  imin=0,/
  irest=1,/
  ntx=5,/
  ntb=2,/
  pres0=1.0,/
  ntp=1,/
  taup=1.0,/
  cut=10.0,/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3,/
  gamma_ln=1.0,/
  ntxo=1,/
  nstlim=100000, dt=0.002,/
  ntpr=100, ntwx=1000, ntwr=1000,/
 /


Md4.in/

Equilibrate md4 for 1 ns in NVT/
 &cntrl/
  imin=0,/
  irest=1, ntx=5,/
  ntb=1,/
  cut=10.0,/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3, gamma_ln=1.0,/
  nstlim=500000, dt=0.002,/
  ntpr=500, ntwx=1000, ntwr=100000,/
 /


Production.in/

Production run NVT/
 &cntrl/
  imin=0,/
  irest=1, ntx=5,/
  ntb=1,/
  ig=-1,/
  cut=10.0,/
  ntc=2, ntf=2,/
  tempi=300.0, temp0=300.0,/
  ntt=3, gamma_ln=1.0,/
  nstlim=50000000, dt=0.002,/
  ntpr=1000, ntwx=500, ntwr=10000,/
 /

