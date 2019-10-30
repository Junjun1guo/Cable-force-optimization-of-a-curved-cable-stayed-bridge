Cable-force-optimization-of-a-curved-cable-stayed-bridge
--------------------------------------------------------
Cable force optimization of a curved cable-stayed bridge with combined simulated annealing 

#### Guo J, Yuan W, Dang X, et al. Cable force optimization of a curved cable-stayed bridge with combined simulated annealing method and cubic B-Spline interpolation curves[J]. Engineering Structures, 2019, 201: 109813.[downLoadPDF](https://www.sciencedirect.com/science/article/pii/S0141029619311046?via%3Dihub)

* [SimulatedAnnealing.py](#SimulatedAnnealingpy)
* [CurvedCableStayedBridge.tcl](#CurvedCableStayedBridgetcl)

![ ](https://github.com/Junjun1guo/Cable-force-optimization-of-a-curved-cable-stayed-bridge/raw/master/paper.png)

SimulatedAnnealing.py
---------------
```python
#-*-coding: UTF-8-*-
################################################################################
#  Author: Junjun Guo
#  E-mail: guojj@tongji.edu.cn/guojj_ce@163.com
#  Date: 20/03/2019
#  Environment: Successfully excucted in python 2.7
################################################################################
import numpy as np
import matplotlib.pyplot as plt
from CoordinateTransform import *
import popen2
import os
import random
import time
from BsplInterpolation import BsplineCurve



def Bspline2CableForce (x_new,forcex,forcey):
	Force1=[]

	for i1 in range(len(x_new)-1):
		for j1 in range(len(forcex)):
			if x_new[i1]<=forcex[j1]:
				Force1.append(forcey[j1])
				break

	Force1.append(forcey[-1])

#	plt.plot(forcex,forcey)
#	plt.plot(x_new,Force1,"o")
#	plt.show()
	return Force1

#######################################################################################################################
def CalculateNewObjectFunction (cableForceList):

	cableEle=np.loadtxt("CableEleForce.txt")
	cableEle[:,3]=cableForceList
	np.savetxt("CableEleForce.txt",cableEle,fmt="%d %d %d %.2f")

	cableNodes=np.loadtxt("CableNode.txt")
	cableEle=np.loadtxt("CableEleForce.txt")
	CableMatEle (cableEle,cableNodes)
#######################################
	a,b =  popen2.popen2('OpenSees.exe')
	b.write("source CurvedCableStayedBridge.tcl\n")
#######################################
#	time.sleep(10)
######################################
	pathDir =os.listdir('GirderResponse/GirderNodeDisp')
	number=[]
	for allDir in pathDir:
		child =  os.path.splitext(allDir)
		number.append(int(child[0]))
	number.sort()

	girderDisp=[]
	for i2 in range(len(number)):
		cwd=os.getcwd()
		pathall=os.path.join(cwd,'GirderResponse/GirderNodeDisp/',str(number[i2]),"1.out")
		txtopen=np.loadtxt(pathall)
		sect=np.mat(txtopen)
		girderDisp.append(sect[0,3])

	absGirderDisp = map(abs, girderDisp)
	maxGirderDisp=max(absGirderDisp)

	squareGirderDisp=map(lambda x: x**2,absGirderDisp)
	sumGirderDisp=sum(squareGirderDisp)



	pathDir1 =  os.listdir('PylonNodeDispSet')
	number1=[]
	for allDir1 in pathDir1:
		child1 =  os.path.splitext(allDir1)
		number1.append(int(child1[0]))
	number1.sort()

	PylonDispx=[]
	PylonDispy=[]
	for i3 in range(len(number1)):
		cwd=os.getcwd()
		pathall=os.path.join(cwd,'PylonNodeDispSet/',str(number1[i3]),"1.out")
		txtopen=np.loadtxt(pathall)
		sect=np.mat(txtopen)
		PylonDispx.append(sect[0,1])
		PylonDispy.append(sect[0,2])

	absPylonDispx = map(abs, PylonDispx)
	absPylonDispy = map(abs, PylonDispy)


	maxPylonDispx=max(absPylonDispx)
	maxPylonDispy=max(absPylonDispy)

	squarePylonDispx=map(lambda x: x**2,absPylonDispx)
	squarePylonDispy=map(lambda x: x**2,absPylonDispy)

	sumPylonDispx=sum(squarePylonDispx)
	sumPylonDispy=sum(squarePylonDispy)

	objectFunction=(sumGirderDisp+sumPylonDispx)**0.5

	return objectFunction, maxGirderDisp,maxPylonDispx,maxPylonDispy

	

########################################################################################################################
def sgnFun (t):
	if t>0:
		return 1
	elif t<0:
		return -1
	else:
		return 0
########################################################################################################################
def yChange (T):
	
	u=random.uniform(0,1)
	y=T*sgnFun(u-0.5)*((1+1/float(T))**abs(2*u-1)-1)
	return y
#######################################################################################################################
def UpdateCableForce (newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4,x_new1,x_new2,x_new3,x_new4):


	forcex1,forcey1=BsplineCurve (newX1,newY1)
#	forcex1,forcey1=BnSpline (newX1,newY1,3,0.01)
	cableForce1=Bspline2CableForce (x_new1,forcex1,forcey1)

	forcex2,forcey2=BsplineCurve (newX2,newY2)
#	forcex2,forcey2=BnSpline (newX2,newY2,3,0.01)
	cableForce2=Bspline2CableForce (x_new2,forcex2,forcey2)

	forcex3,forcey3=BsplineCurve (newX3,newY3)
#	forcex3,forcey3=BnSpline (newX3,newY3,3,0.01)
	cableForce3=Bspline2CableForce (x_new3,forcex3,forcey3)

	forcex4,forcey4=BsplineCurve (newX4,newY4)
#	forcex4,forcey4=BnSpline (newX4,newY4,3,0.01)
	cableForce4=Bspline2CableForce (x_new4,forcex4,forcey4)

	cableForceList=cableForce1+cableForce1+cableForce2+cableForce2+cableForce3+cableForce3+cableForce4+cableForce4

	return cableForceList
##########################################################################################################################
def UpdateCoordinatesValue (T,initX1,initX2,initX3,initX4,initY1,initY2,initY3,initY4):
	newX1=[initX1[0],initX1[1]+yChange (T)*(24-24),initX1[2]+yChange (T)*(48-48),initX1[3]]
	newX2=[initX2[0],initX2[1]+yChange (T)*(40-40),initX2[2]+yChange (T)*(80-80),initX2[3]]
	newX3=[initX3[0],initX3[1]+yChange (T)*(60-60),initX3[2]+yChange (T)*(120-120),initX3[3]]
	newX4=[initX4[0],initX4[1]+yChange (T)*(35-35),initX4[2]+yChange (T)*(70-70),initX4[3]]

	
	while True:
		newY1=[initY1[0]+yChange (T)*1000,initY1[1]+yChange (T)*1000,initY1[2]+yChange (T)*1000,initY1[3]+yChange (T)*1000]
		if 1000<newY1[0]<newY1[1]<newY1[2]<newY1[3]<5500:
			break
		else:
			continue
	while True:
		newY2=[initY2[0]+yChange (T)*1000,initY2[1]+yChange (T)*1000,initY2[2]+yChange (T)*1000,initY2[3]+yChange (T)*1000]
		if 1000<newY2[0]<newY2[1]<newY2[2]<newY2[3]<5500:
			break
		else:
			continue
	while True:
		newY3=[initY3[0]+yChange (T)*1000,initY3[1]+yChange (T)*1000,initY3[2]+yChange (T)*1000,initY3[3]+yChange (T)*1000]
		if 1000<newY3[0]<newY3[1]<newY3[2]<newY3[3]<5500:
			break
		else:
			continue
	while True:
		newY4=[initY4[0]+yChange (T)*1000,initY4[1]+yChange (T)*1000,initY4[2]+yChange (T)*1000,initY4[3]+yChange (T)*1000]
		if 1000<newY4[0]<newY4[1]<newY4[2]<newY4[3]<5500:
			break
		else:
			continue

	return newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4
	
##########################################################################################################################

def SA (maxT,minT,nIter,initX1,initX2,initX3,initX4,initY1,initY2,initY3,initY4,x_new1,x_new2,x_new3,x_new4):

	T=maxT
	k=0
	savenewmaxGDisp=[]
	savenewmaxPyDispx=[]
	savenewmaxPyDispy=[]
	savenewE=[]
	savecableForceList=[]
	savek=[]

	
	
	while T>=minT:

		kk=0
		savekk=[]

		for iiter in range(nIter):

			oldCableForceList=UpdateCableForce (initX1,initX2,initX3,initX4,initY1,initY2,initY3,initY4,x_new1,x_new2,x_new3,x_new4)


			while True:
				try:
					oldE, oldMaxGDisp,oldMaxPyDispx,oldMaxPyDispy=CalculateNewObjectFunction (oldCableForceList)
					break
				except IndexError:
					break


			
			newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4=UpdateCoordinatesValue (T,initX1,initX2,initX3,initX4,initY1,initY2,initY3,initY4)
			newCableForceList=UpdateCableForce (newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4,x_new1,x_new2,x_new3,x_new4)

			while True:
				try:
					newE,newMaxGDisp,newMaxPyDispx,newMaxPyDispy=CalculateNewObjectFunction (newCableForceList)
					break
				except IndexError:
					newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4=UpdateCoordinatesValue (T,newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4)
					newCableForceList=UpdateCableForce (newX1,newX2,newX3,newX4,newY1,newY2,newY3,newY4,x_new1,x_new2,x_new3,x_new4)

			deltaE=newE-oldE

			if deltaE<0:
				initX1=newX1
				initX2=newX2
				initX3=newX3
				initX4=newX4

				initY1=newY1
				initY2=newY2
				initY3=newY3
				initY4=newY4

			else:
				p=np.exp(-deltaE/float(T))
				if p>(random.uniform(0,1)):
					initX1=newX1
					initX2=newX2
					initX3=newX3
					initX4=newX4

					initY1=newY1
					initY2=newY2
					initY3=newY3
					initY4=newY4

			kk=kk+1
			savekk.append(kk)

			np.savetxt("iterInternal.txt",savekk,fmt="%d")
###################################################################################################################
#		
		savenewmaxGDisp.append(oldMaxGDisp)
		savenewmaxPyDispx.append(oldMaxPyDispx)
		savenewmaxPyDispy.append(oldMaxPyDispy)
		savenewE.append(oldE)
		savecableForceList.append(oldCableForceList)
		savek.append(k)

		
		np.savetxt("maxGirderDisp.txt",savenewmaxGDisp,fmt="%.6f")

		np.savetxt("maxPylonDispx.txt",savenewmaxPyDispx,fmt="%.6f")

		np.savetxt("maxPylonDispy.txt",savenewmaxPyDispy,fmt="%.6f")

		np.savetxt("objectFunctionValue.txt",savenewE,fmt="%.6f")

		np.savetxt("iterCableForceValue.txt",savecableForceList,fmt="%.2f")                                        

		np.savetxt("iterNumber.txt",savek,fmt="%d")

		k=k+1
		alpha=0.95 #alpha [0.7 1]
		T=maxT*alpha**k


################################################# Main program ##############################
x1Initial=[0.00,24.00,48.00,73.15]
x2Initial=[0.00,40.00,80.00,122.74]
x3Initial=[0.00,60.00,120.00,192.00]
x4Initial=[0.00,35.00,70.00,117.42]
y1Initial=[1000,2500,4000,5500]
y2Initial=[1000,2500,4000,5500]
y3Initial=[1000,2500,4000,5500]
y4Initial=[1000,2500,4000,5500]
x_new1=[0,7.83,15.63,23.4,31.15,38.88,46.57,54.23,61.86,67.51,73.15]#1101-1111
x_new2=[0.00,12.21,24.46,36.74,49.04,61.36,73.69,86.01,98.30,110.55,122.74 ]#1301-1311
x_new3=[0.00,11.99,23.98,35.96,47.95,59.94,71.93,83.92,95.91,107.89,119.88,131.87,143.86,155.85,167.86,179.91,192.00]#1501-1517
x_new4=[0.00,7.79,15.58,23.38,31.10,38.90,46.69,54.48,62.27,70.07,77.86,85.65,93.44,101.11,108.77,113.09,117.42]#1701-1717
####################################################################
maxT=20
minT=1e-5
nIter=2

SA(maxT,minT,nIter,x1Initial,x2Initial,x3Initial,x4Initial,y1Initial,y2Initial,y3Initial,y4Initial,x_new1,x_new2,x_new3,x_new4)
```

CurvedCableStayedBridge.tcl
---------------------------
```tcl
######Curved Cable-stayed bridge model
######Units: Length-m, Force-kN, mass-ton, Stress-kpa(10e-3MPa), g=9.81m/s2

wipe
###########################################################################################

model basic -ndm 3 -ndf 6

####################################--CableNodes--#########################################
puts "Building the cable node..."
set Data [open newCableNodes.txt r]
set var3 [split [read $Data]]
set CableNP [expr  [llength $var3]/5]

for {set i 1} {$i<=$CableNP} {incr i 1} {
	set NodeCable($i)     	[lindex	$var3	[expr 5*($i-1)]]
	set XCable($i)       	[lindex 	$var3 [expr 5*($i-1)+1]]
	set YCable($i)       	[lindex 	$var3 [expr 5*($i-1)+2]]
	set ZCable($i)       	[lindex 	$var3 [expr 5*($i-1)+3]]
	set NodeMCable($i) 	[lindex 	$var3 [expr 5*($i-1)+4]] 


	node $NodeCable($i) $XCable($i) 	$YCable($i) 	$ZCable($i)
	mass $NodeCable($i) $NodeMCable($i) $NodeMCable($i) 	$NodeMCable($i) 0.0 0.0 0.0
#	puts "$NodeCable($i) $XCable($i) 	$YCable($i) 	$ZCable($i)		$NodeMCable($i)"
}
close $Data
puts "Cable node building is completed..."
#############################################################################################

#######################################---CableMaterial---###################################
puts "Building Cable Material..."
set Data [open newCableMat.txt r]
set var3 [split [read $Data]]
set CableNMatN [expr  [llength $var3]/3]
for {set i 1} {$i<=$CableNMatN} {incr i 1} {
	set MatCable($i)	[lindex 	$var3 [expr 3*($i-1)]]
	set ECable($i)   	[lindex 	$var3 [expr 3*($i-1)+1]]
	set PreStr($i)  	[lindex 	$var3 [expr 3*($i-1)+2]]
#	puts "$MatCable($i)  $ECable($i) $PreStr($i)"
	set eps0($i)		[expr -$PreStr($i)/$ECable($i)]
	set epsyN($i)		0
	set epsyP($i)		[expr 1.77e6/$ECable($i)+$eps0($i)]				
	uniaxialMaterial ElasticPP 		$MatCable($i) 	$ECable($i) 	$epsyP($i)		$epsyN($i)		$eps0($i)	
}
close $Data
###########################################################################################

##################################---CableElement---#######################################
puts "Building cable element..."
set Data [open newCableEle.txt r]
set var3 [split [read $Data]]
set CableEleN [expr  [llength $var3]/5]
for {set i 1} {$i<=$CableEleN} {incr i 1} {
	set EleCable($i)  	[lindex $var3 [expr 5*($i-1)]]
	set NodeICable($i)    	[lindex $var3 [expr 5*($i-1)+1]]
	set NodeJCable($i)    	[lindex $var3 [expr 5*($i-1)+2]]
	set ACable($i)		[lindex $var3 [expr 5*($i-1)+3]]
	set MatCable($i) 		[lindex $var3 [expr 5*($i-1)+4]]

	element truss $EleCable($i) $NodeICable($i) $NodeJCable($i) $ACable($i) $MatCable($i)
#	puts "$EleCable($i)	$NodeICable($i)	$NodeJCable($i)	$ACable($i)	$MatCable($i)"
}

puts "Cable element building is completed"
close $Data
############################################################################################

##################################---GirderNode---###########################################
puts "Building the girder node..."
set Data 	[open newGirderNodes.txt r]
set var3 	[split [read $Data]]
set GirderNP [expr  [llength $var3]/5]

for {set i 1} {$i<=$GirderNP} {incr i 1} {
	set NodeGirder($i)     	[lindex $var3 [expr 5*($i-1)]]
	set XGirder($i)       	[lindex $var3 [expr 5*($i-1)+1]]
	set YGirder($i)       	[lindex $var3 [expr 5*($i-1)+2]]
	set ZGirder($i)       	[lindex $var3 [expr 5*($i-1)+3]]
	set NodeMGirder($i) 	[lindex $var3 [expr 5*($i-1)+4]] 

	node $NodeGirder($i) 	$XGirder($i) 	$YGirder($i) 	$ZGirder($i)
	mass $NodeGirder($i) 	$NodeMGirder($i) 	$NodeMGirder($i) 	$NodeMGirder($i) 0.0 0.0 0.0
	
#	puts "$NodeGirder($i)	$XGirder($i)	$YGirder($i)	$ZGirder($i)	$NodeMGirder($i)"
}
close $Data
puts "Girder node building is completed..."
#############################################################################################

############################---GirderElementLocalAxis---#####################################
puts "Building the girder local axis transform..."
set Data 	[open newGirderTransf.txt r]
set var3 	[split [read $Data]]
set GirderAxisNum [expr  [llength $var3]/4]
for {set i 1} {$i<=$GirderAxisNum} {incr i 1} {
	set GirderTransN($i)     	[lindex $var3 [expr 4*($i-1)]]
	set XCoor($i)       		[lindex $var3 [expr 4*($i-1)+1]]
	set YCoor($i)       		[lindex $var3 [expr 4*($i-1)+2]]
	set ZCoor($i)       		[lindex $var3 [expr 4*($i-1)+3]]
	
	geomTransf PDelta $GirderTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)
	
#	puts "$GirderTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)"
}
close $Data
puts "Girder local axis transform building is completed..."
#############################################################################################

##########################---GirderElement---################################################
puts "Building the girder element..."
set Data [open GirderEle.txt r]
set var3 [split [read $Data]]
set GirderEleNum [expr  [llength $var3]/10]
for {set i 1} {$i<=$GirderEleNum} {incr i 1} {
	set EleGirder($i) 	[lindex $var3 [expr 10*($i-1)]]
	set NodeIGirder($i)  	[lindex $var3 [expr 10*($i-1)+1]]
	set NodeJGirder($i)  	[lindex $var3 [expr 10*($i-1)+2]]
	set AGirder($i)      	[lindex $var3 [expr 10*($i-1)+3]]
	set EGirder($i)		[lindex $var3 [expr 10*($i-1)+4]]
	set GGirder($i)		[lindex $var3 [expr 10*($i-1)+5]]
	set JGirder($i)     	[lindex $var3 [expr 10*($i-1)+6]]
	set IyGirder($i)     	[lindex $var3 [expr 10*($i-1)+7]]
	set IzGirder($i)      	[lindex $var3 [expr 10*($i-1)+8]]
	set GirderTran($i)	[lindex $var3 [expr 10*($i-1)+9]]

	element elasticBeamColumn $EleGirder($i) $NodeIGirder($i) $NodeJGirder($i) $AGirder($i) $EGirder($i) $GGirder($i) $JGirder($i) $IyGirder($i) $IzGirder($i) $GirderTran($i)

#	puts "$EleGirder($i) $NodeIGirder($i) $NodeJGirder($i) $AGirder($i) $EGirder($i) $GGirder($i) $JGirder($i) $IyGirder($i) $IzGirder($i) $GirderTran($i)"
}
close $Data
puts "Girder element building is completed..."
#############################################################################################

################################---PylonNode---##############################################
puts "Building the pylon node..."
set Data 	[open newPylonNodes.txt r]
set var3 	[split [read $Data]]
set PylonNP [expr  [llength $var3]/5]

for {set i 1} {$i<=$PylonNP} {incr i 1} {
	set NodePylon($i)     	[lindex $var3 [expr 5*($i-1)]]
	set XPylon($i)       	[lindex $var3 [expr 5*($i-1)+1]]
	set YPylon($i)       	[lindex $var3 [expr 5*($i-1)+2]]
	set ZPylon($i)       	[lindex $var3 [expr 5*($i-1)+3]]
	set NodeMPylon($i) 	[lindex $var3 [expr 5*($i-1)+4]] 

	node $NodePylon($i) 	$XPylon($i) 	$YPylon($i) 	$ZPylon($i)
	mass $NodePylon($i) 	$NodeMPylon($i) 	$NodeMPylon($i) 	$NodeMPylon($i) 0.0 0.0 0.0	
#	puts "$NodePylon($i)	$XPylon($i)		$YPylon($i)		$ZPylon($i)		$NodeMPylon($i)"
}

close $Data
puts "Pylon node building is completed..."
#############################################################################################

######################################---PylonFiber---#######################################
puts "Building the fiber pylon element..."
set Data [open PylonSec.txt r]
set var3 [split [read $Data]]
source FiberDivide.tcl
set 		a			0.05
set 		CoverC50		$C50_Cover(1)
set 		CoreC50		$C50_Core(1)
set		BarHRB400		$HRB400(1)
set		CoverC40		$C40_Cover(1)
set		CoreC40		$C40_Core(1)
set		PylonFiberWidth	0.2
set 		StellDelta		0.15

set GJ 1.0e10; 
uniaxialMaterial 	Elastic   20	$GJ;         # Define torsional stiffness
###################--PylonSection1--####################
for {set i 1} {$i<=10} {incr i 1} {
	set 	PylonEle 		[lindex $var3 [expr 12*($i-1)]]
	set 	L			[lindex $var3 [expr 12*($i-1)+1]]
	set 	W			[lindex $var3 [expr 12*($i-1)+2]]
	set 	L1 			[lindex $var3 [expr 12*($i-1)+3]]
	set 	L2			[lindex $var3 [expr 12*($i-1)+4]]
	set 	L3			[lindex $var3 [expr 12*($i-1)+5]]
	set 	L4			[lindex $var3 [expr 12*($i-1)+6]]
	set 	L5			[lindex $var3 [expr 12*($i-1)+7]]
	set 	L6			[lindex $var3 [expr 12*($i-1)+8]]
	set 	L7			[lindex $var3 [expr 12*($i-1)+9]]
	set 	W1			[lindex $var3 [expr 12*($i-1)+10]]
	set 	Row			[lindex $var3 [expr 12*($i-1)+11]]
# Rect3Hole {SecTag L W L1 L2 L3 L4 L5 L6 L7 W1 a row delta fiberWidth Cover Core Bar}
	Rect3Hole $PylonEle $L $W $L1 $L2 $L3 $L4 $L5 $L6 $L7 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400
	set PylonSec($i)	[expr 10000+$PylonEle]
	section Aggregator $PylonSec($i)  20 T -section $PylonEle;
#	 puts "$PylonEle $L $W $L1 $L2 $L3 $L4 $L5 $L6 $L7 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400"
}
####################--PylonSection2--#######################
for {set i 11} {$i<=76} {incr i 1} {
	set 	PylonEle		[lindex $var3 [expr 120+6*($i-11)]]
	set 	L			[lindex $var3 [expr 120+6*($i-11)+1]]
	set 	W			[lindex $var3 [expr 120+6*($i-11)+2]]
	set 	L1			[lindex $var3 [expr 120+6*($i-11)+3]]
	set 	W1			[lindex $var3 [expr 120+6*($i-11)+4]]
	set 	Row			[lindex $var3 [expr 120+6*($i-11)+5]]
#Rect1Hole {SecTag L W L1 W1 a row delta fiberWidth Cover Core Bar}		
	Rect1Hole $PylonEle $L $W $L1 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400
	set PylonSec($i)	[expr 10000+$PylonEle]
	section Aggregator $PylonSec($i)  20 T 	-section $PylonEle;
# 	puts "$PylonEle $L $W $L1 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400"
}
##################--PylonSection3--#########################
for {set i 77} {$i<=88} {incr i 1} {
	set 	PylonEle 		[lindex $var3 [expr 516+12*($i-77)]]
	set 	L			[lindex $var3 [expr 516+12*($i-77)+1]]
	set 	W			[lindex $var3 [expr 516+12*($i-77)+2]]
	set 	L1 			[lindex $var3 [expr 516+12*($i-77)+3]]
	set 	L2			[lindex $var3 [expr 516+12*($i-77)+4]]
	set 	L3			[lindex $var3 [expr 516+12*($i-77)+5]]
	set 	L4			[lindex $var3 [expr 516+12*($i-77)+6]]
	set 	L5			[lindex $var3 [expr 516+12*($i-77)+7]]
	set 	L6			[lindex $var3 [expr 516+12*($i-77)+8]]
	set 	L7			[lindex $var3 [expr 516+12*($i-77)+9]]
	set 	W1			[lindex $var3 [expr 516+12*($i-77)+10]]
	set 	Row			[lindex $var3 [expr 516+12*($i-77)+11]]
# Rect3Hole {SecTag L W L1 L2 L3 L4 L5 L6 L7 W1 a row delta fiberWidth Cover Core Bar}
	Rect3Hole $PylonEle $L $W $L1 $L2 $L3 $L4 $L5 $L6 $L7 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400
	set PylonSec($i)	[expr 10000+$PylonEle]
	section Aggregator $PylonSec($i)  20 T -section $PylonEle;
#	 puts "$PylonEle $L $W $L1 $L2 $L3 $L4 $L5 $L6 $L7 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400"
}
#################--PylonSection4--#########################
for {set i 89} {$i<=180} {incr i 1} {
	set 	PylonEle		[lindex $var3 [expr 660+6*($i-89)]]
	set 	L			[lindex $var3 [expr 660+6*($i-89)+1]]
	set 	W			[lindex $var3 [expr 660+6*($i-89)+2]]
	set 	L1			[lindex $var3 [expr 660+6*($i-89)+3]]
	set 	W1			[lindex $var3 [expr 660+6*($i-89)+4]]
	set 	Row			[lindex $var3 [expr 660+6*($i-89)+5]]
#Rect1Hole {SecTag L W L1 W1 a row delta fiberWidth Cover Core Bar}		
	Rect1Hole $PylonEle $L $W $L1 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400
	set PylonSec($i)	[expr 10000+$PylonEle]
	section Aggregator $PylonSec($i)  20 T 	-section $PylonEle;
# 	puts "$PylonEle $L $W $L1 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400"
}
##################--PylonSection5--######################
for {set i 181} {$i<=191} {incr i 1} {
	set 	PylonEle 		[lindex $var3 [expr 1212+12*($i-181)]]
	set 	L			[lindex $var3 [expr 1212+12*($i-181)+1]]
	set 	W			[lindex $var3 [expr 1212+12*($i-181)+2]]
	set 	L1 			[lindex $var3 [expr 1212+12*($i-181)+3]]
	set 	L2			[lindex $var3 [expr 1212+12*($i-181)+4]]
	set 	L3			[lindex $var3 [expr 1212+12*($i-181)+5]]
	set 	L4			[lindex $var3 [expr 1212+12*($i-181)+6]]
	set 	L5			[lindex $var3 [expr 1212+12*($i-181)+7]]
	set 	L6			[lindex $var3 [expr 1212+12*($i-181)+8]]
	set 	L7			[lindex $var3 [expr 1212+12*($i-181)+9]]
	set 	W1			[lindex $var3 [expr 1212+12*($i-181)+10]]
	set 	Row			[lindex $var3 [expr 1212+12*($i-181)+11]]
# Rect3Hole {SecTag L W L1 L2 L3 L4 L5 L6 L7 W1 a row delta fiberWidth Cover Core Bar}
	Rect3Hole $PylonEle $L $W $L1 $L2 $L3 $L4 $L5 $L6 $L7 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400
	set PylonSec($i)	[expr 10000+$PylonEle]
	section Aggregator $PylonSec($i)  20 T -section $PylonEle;
#	 puts "$PylonEle $L $W $L1 $L2 $L3 $L4 $L5 $L6 $L7 $W1 $a $Row $StellDelta $PylonFiberWidth $CoverC50 $CoreC50 $BarHRB400"
}
################--PierSection--#########################
for {set i 192} {$i<=225} {incr i 1} {
	set 	PylonEle		[lindex $var3 [expr 1344+4*($i-192)]]
	set 	L			[lindex $var3 [expr 1344+4*($i-192)+1]]
	set 	W			[lindex $var3 [expr 1344+4*($i-192)+2]]
	set 	Row			[lindex $var3 [expr 1344+4*($i-192)+3]]
#Rect {SecTag L W a row delta fiberWidth Cover Core Bar}		
	Rect $PylonEle $L $W $a $Row $StellDelta $PylonFiberWidth $CoverC40 $CoreC40 $BarHRB400
	set PylonSec($i)	[expr 10000+$PylonEle]
	section Aggregator $PylonSec($i)  20 T 	-section $PylonEle;
# 	puts "$PylonEle $L $W $a $Row $StellDelta $PylonFiberWidth $CoverC40 $CoreC40 $BarHRB400"
}
close $Data
########################################################

#################################---PierTransf---############################################
puts "Building the pier local axis transform..."
set Data 	[open newPierTransf.txt r]
set var3 	[split [read $Data]]
set PierAxisNum [expr  [llength $var3]/4]
for {set i 1} {$i<=$PierAxisNum} {incr i 1} {
	set PierTransN($i)     	[lindex $var3 [expr 4*($i-1)]]
	set XCoor($i)       		[lindex $var3 [expr 4*($i-1)+1]]
	set YCoor($i)       		[lindex $var3 [expr 4*($i-1)+2]]
	set ZCoor($i)       		[lindex $var3 [expr 4*($i-1)+3]]
	
	geomTransf PDelta $PierTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)
	
#	puts "$PierTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)"
}
close $Data
puts "Pier local axis transform building is completed..."
#############################################################################################

########################################################
set Data [open PylonEle.txt r]
set var3 [split [read $Data]]
set PylonNEle [expr  [llength $var3]/4]
set np   5
for {set i 1} {$i<=$PylonNEle} {incr i 1} {
	set ElePylon($i) 		[lindex $var3 [expr 4*($i-1)]]
	set NodeIPylon($i)	[lindex $var3 [expr 4*($i-1)+1]]
	set NodeJPylon($i)	[lindex $var3 [expr 4*($i-1)+2]]
      set GeomTrans($i)	      [lindex $var3 [expr 4*($i-1)+3]]
	element nonlinearBeamColumn $ElePylon($i) $NodeIPylon($i) $NodeJPylon($i) $np $PylonSec($i) $GeomTrans($i)

#	puts "$ElePylon1($i) $NodeIPylon($i) $NodeJPylon($i)	$PylonSec($i)"
}
close $Data
###################################################################################################

##################################---CrossBeamNode---##############################################
puts "Building the CrossBeam node..."
set Data 	[open newCrossBeamNodes.txt r]
set var3 	[split [read $Data]]
set CrossBeamN [expr  [llength $var3]/5]
for {set i 1} {$i<=$CrossBeamN} {incr i 1} {
	set NodeCrBm($i)     	[lindex $var3 [expr 5*($i-1)]]
	set XCrBm($i)       	[lindex $var3 [expr 5*($i-1)+1]]
	set YCrBm($i)       	[lindex $var3 [expr 5*($i-1)+2]]
	set ZCrBm($i)       	[lindex $var3 [expr 5*($i-1)+3]]
	set NodeMCrBm($i) 	[lindex $var3 [expr 5*($i-1)+4]] 

	node $NodeCrBm($i) 	$XCrBm($i) 		$YCrBm($i) 		$ZCrBm($i)
	mass $NodeCrBm($i) 	$NodeMCrBm($i) 	$NodeMCrBm($i) 	$NodeMCrBm($i) 0.0 0.0 0.0
#	puts "$NodeCrBm($i)	$XCrBm($i)	$YCrBm($i)	$ZCrBm($i)	$NodeMCrBm($i)"
}
close $Data
puts "CrossBeam node building is completed..."
###################################################################################################

#####################################---CrossBeamTransf---#########################################
puts "Building the crossBeam local axis transform..."
set Data 	[open newCrossBeamTransf.txt r]
set var3 	[split [read $Data]]
set CrossBeamTransNum [expr  [llength $var3]/4]
for {set i 1} {$i<=$CrossBeamTransNum} {incr i 1} {
	set CrossBeamTransN($i)     	[lindex $var3 [expr 4*($i-1)]]
	set XCoor($i)       		[lindex $var3 [expr 4*($i-1)+1]]
	set YCoor($i)       		[lindex $var3 [expr 4*($i-1)+2]]
	set ZCoor($i)       		[lindex $var3 [expr 4*($i-1)+3]]
	
	geomTransf PDelta $CrossBeamTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)
	
#	puts "$CrossBeamTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)"
}
close $Data
puts "crossBeam local axis transform building is completed..."
###################################################################################################

#######################################---CrossElement---##########################################
puts "Building the crossbeam element..."
set Data [open CrossBeamEle.txt r]
set var3 [split [read $Data]]
set CrossBeamEleN [expr  [llength $var3]/8]

set ECrBm	3.25e7
set GCrBm	1.35e7

for {set i 1} {$i<=$CrossBeamEleN} {incr i 1} {
	set EleCrBm($i) 		[lindex $var3 [expr 8*($i-1)]]
	set NodeICrBm($i)  	[lindex $var3 [expr 8*($i-1)+1]]
	set NodeJCrBm($i)  	[lindex $var3 [expr 8*($i-1)+2]]
	set ACrBm($i)      	[lindex $var3 [expr 8*($i-1)+3]]
	set JCrBm($i)     	[lindex $var3 [expr 8*($i-1)+4]]
	set IyCrBm($i)     	[lindex $var3 [expr 8*($i-1)+5]]
	set IzCrBm($i)      	[lindex $var3 [expr 8*($i-1)+6]]
	set GeomTransf($i)	[lindex $var3 [expr 8*($i-1)+7]]
	element elasticBeamColumn $EleCrBm($i) $NodeICrBm($i) $NodeJCrBm($i) $ACrBm($i) $ECrBm $GCrBm $JCrBm($i) $IyCrBm($i) $IzCrBm($i) $GeomTransf($i)
#	puts "$EleCrBm($i) $NodeICrBm($i) $NodeJCrBm($i)"
}
close $Data
puts "CrossBeam element building is completed..."
##################################################################################################

########################################---OtherNodes---##########################################
puts "Building the other node..."
set Data 	[open newOtherNodes.txt r]
set var3 	[split [read $Data]]
set OtherNP [expr  [llength $var3]/5]

for {set i 1} {$i<=$OtherNP} {incr i 1} {
	set NodeOther($i)     	[lindex $var3 [expr 5*($i-1)]]
	set XOther($i)       	[lindex $var3 [expr 5*($i-1)+1]]
	set YOther($i)       	[lindex $var3 [expr 5*($i-1)+2]]
	set ZOther($i)       	[lindex $var3 [expr 5*($i-1)+3]]
	set NodeMOther($i) 	[lindex $var3 [expr 5*($i-1)+4]] 

	node $NodeOther($i) 	$XOther($i) 	$YOther($i) 	$ZOther($i)
	mass $NodeOther($i) 	$NodeMOther($i) 	$NodeMOther($i) 	$NodeMOther($i) 0.0 0.0 0.0
	
#	puts "$NodeOther($i)	$XOther($i)	$YOther($i) $ZOther($i)	$NodeMOther($i)"
}
close $Data
puts "Other node building is completed..."
##################################################################################################

#######################################---EqualDOFTransf---#######################################
puts "Building the equalDOF local axis transform..."
set Data 	[open newEqualDOFTransf.txt r]
set var3 	[split [read $Data]]
set equalDOFNum [expr  [llength $var3]/3]
for {set i 1} {$i<=$equalDOFNum} {incr i 1} {
	set XCoor($i)       		[lindex $var3 [expr 3*($i-1)+0]]
	set YCoor($i)       		[lindex $var3 [expr 3*($i-1)+1]]
	set ZCoor($i)       		[lindex $var3 [expr 3*($i-1)+2]]
	
	geomTransf PDelta 200001 $XCoor($i) $YCoor($i) $ZCoor($i)
	
#	puts "$CrossBeamTransN($i) $XCoor($i) $YCoor($i) $ZCoor($i)"
}
close $Data
puts "equalDOF local axis transform building is completed..."
##################################################################################################

######################################---EqualDOF---##############################################
puts "Building equalDOF..."
set Data [open EqualDoF.txt r]
set var3 [split [read $Data]]
set EqualN [expr  [llength $var3]/10]

for {set i 1} {$i<=$EqualN} {incr i 1} {
	set EleGirder($i) 	[lindex $var3 [expr 10*($i-1)]]
      set NodeIGirder($i) 	[lindex $var3 [expr 10*($i-1)+1]]
	set NodeJGirder($i) 	[lindex $var3 [expr 10*($i-1)+2]]
	set AGirder($i) 		[lindex $var3 [expr 10*($i-1)+3]]
	set EGirder($i) 		[lindex $var3 [expr 10*($i-1)+4]]
	set GGirder($i) 		[lindex $var3 [expr 10*($i-1)+5]]
	set JGirder($i) 		[lindex $var3 [expr 10*($i-1)+6]]
	set IyGirder($i) 		[lindex $var3 [expr 10*($i-1)+7]]
	set IzGirder($i) 		[lindex $var3 [expr 10*($i-1)+8]]
	set Gemtransf($i) 	[lindex $var3 [expr 10*($i-1)+9]]
	element elasticBeamColumn $EleGirder($i) $NodeIGirder($i) $NodeJGirder($i) $AGirder($i) $EGirder($i) $GGirder($i) $JGirder($i) $IyGirder($i) $IzGirder($i) $Gemtransf($i)
}
close $Data
puts "equalDOF building is completed..."
###############################################################################################

#####################################---Bearing---###################################################
uniaxialMaterial ENT 9001 1E10
uniaxialMaterial Elastic 9002 1E10 
########################################################

set Data 	[open newBearingTransf.txt r]
set var3 	[split [read $Data]]
set 	BearingNum [expr  [llength $var3]/3]
for {set i 1} {$i<=$BearingNum} {incr i 1} {
	set YxCoor($i)       		[lindex $var3 [expr 3*($i-1)+0]]
	set YyCoor($i)       		[lindex $var3 [expr 3*($i-1)+1]]
	set YzCoor($i)       		[lindex $var3 [expr 3*($i-1)+2]]
	
}
close $Data
#####################################################
	element zeroLength 39201 9202 9201 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(1) $YyCoor(1) $YzCoor(1)	
	element zeroLength 39203 9204 9203 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(1) $YyCoor(1) $YzCoor(1)

	element zeroLength 39301 9302 9301 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(2) $YyCoor(2) $YzCoor(2)	
	element zeroLength 39303 9304 9303 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(2) $YyCoor(2) $YzCoor(2)

	element zeroLength 39401 4207 9401 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(3) $YyCoor(3) $YzCoor(3)	
	element zeroLength 39402 4307 9402 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(3) $YyCoor(3) $YzCoor(3)

	element zeroLength 39501 5207 9501 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(4) $YyCoor(4) $YzCoor(4)	
	element zeroLength 39502 5307 9502 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(4) $YyCoor(4) $YzCoor(4)

	element zeroLength 39601 6203 9601 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(5) $YyCoor(5) $YzCoor(5)	
	element zeroLength 39602 6303 9602 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(5) $YyCoor(5) $YzCoor(5)

	element zeroLength 39701 7203 9701 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(6) $YyCoor(6) $YzCoor(6)	
	element zeroLength 39702 7303 9702 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(6) $YyCoor(6) $YzCoor(6)

	element zeroLength 39801 8202 9801 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(7) $YyCoor(7) $YzCoor(7)	
	element zeroLength 39802 8302 9802 -mat 9001 9002 -dir 1 3 -orient 0 0 1 $YxCoor(7) $YyCoor(7) $YzCoor(7)
###########################################################################################################

###########################################---LinerSpring---###############################################
uniaxialMaterial Elastic 40001 3.7e7
uniaxialMaterial Elastic 40002 1.0e7
uniaxialMaterial Elastic 40003 2.0e8
uniaxialMaterial Elastic 40004 4.0e8
uniaxialMaterial Elastic 40005 2.5e7
uniaxialMaterial Elastic 40006 7.2e6
uniaxialMaterial Elastic 40007 7.0e7
uniaxialMaterial Elastic 40008 9.5e7
uniaxialMaterial Elastic 40009 2.7e8
uniaxialMaterial Elastic 40010 1.2e8
uniaxialMaterial Elastic 40011 6.4e7
uniaxialMaterial Elastic 40012 5.4e7
uniaxialMaterial Elastic 40013 5.1e7
uniaxialMaterial Elastic 40014 1.0e10
uniaxialMaterial Elastic 40015 1.1e10
uniaxialMaterial Elastic 40016 1.9e9
uniaxialMaterial Elastic 40017 7.2e7
uniaxialMaterial Elastic 40018 6.1e7
uniaxialMaterial Elastic 40019 6.0e7
uniaxialMaterial Elastic 40020 1.6e10
uniaxialMaterial Elastic 40021 1.8e10
uniaxialMaterial Elastic 40022 2.4e9

element zeroLength 43001 43001 13103 -mat 40011 40012 40013 40014 40015 40016  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(1) $YyCoor(1) $YzCoor(1)

element zeroLength 44001 44001 14102 -mat 40017 40018 40019 40020 40021 40022  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(2) $YyCoor(2) $YzCoor(2)

element zeroLength 41001 41001 11102 -mat 40001 40002 40002 40003 40004 40004  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(3) $YyCoor(3) $YzCoor(3)
element zeroLength 41002 41002 11202 -mat 40001 40002 40002 40003 40004 40004  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(3) $YyCoor(3) $YzCoor(3)

element zeroLength 42001 42001 12102 -mat 40005 40006 40007 40008 40009 40010  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(4) $YyCoor(4) $YzCoor(4)
element zeroLength 42002 42002 12202 -mat 40005 40006 40007 40008 40009 40010  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(4) $YyCoor(4) $YzCoor(4)

element zeroLength 45001 45001 15102 -mat 40005 40006 40007 40008 40009 40010  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(5) $YyCoor(5) $YzCoor(5)
element zeroLength 45002 45002 15202 -mat 40005 40006 40007 40008 40009 40010  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(5) $YyCoor(5) $YzCoor(5)

element zeroLength 46001 46001 16102 -mat 40005 40006 40007 40008 40009 40010  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(6) $YyCoor(6) $YzCoor(6)
element zeroLength 46002 46002 16202 -mat 40005 40006 40007 40008 40009 40010  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(6) $YyCoor(6) $YzCoor(6)

element zeroLength 47001 47001 17102 -mat 40001 40002 40002 40003 40004 40004  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(7) $YyCoor(7) $YzCoor(7)
element zeroLength 47002 47002 17202 -mat 40001 40002 40002 40003 40004 40004  -dir 1 2 3 4 5 6 -orient 0 0 1 $YxCoor(7) $YyCoor(7) $YzCoor(7)
###########################################################################################################

#############################################---fix---#####################################################
fix 41001 1 1 1 1 1 1
fix 41002 1 1 1 1 1 1

fix 42001 1 1 1 1 1 1
fix 42002 1 1 1 1 1 1

fix 43001 1 1 1 1 1 1

fix 44001 1 1 1 1 1 1

fix 45001 1 1 1 1 1 1
fix 45002 1 1 1 1 1 1

fix 46001 1 1 1 1 1 1
fix 46002 1 1 1 1 1 1

fix 47001 1 1 1 1 1 1
fix 47002 1 1 1 1 1 1
###############################################################################################

###############################################################################################
set g 9.81
pattern Plain 1 Linear {
	for {set i 1} {$i<=$CableNP} {incr i 1} {
     		load $NodeCable($i) 0.0 0.0 [expr -$NodeMCable($i)*$g] 0.0 0.0 0.0 
    	}
	for {set i 1} {$i<=$GirderNP} {incr i 1} {
     		load $NodeGirder($i) 0.0 0.0 [expr -$NodeMGirder($i)*$g] 0.0 0.0 0.0 
    	}
	for {set i 1} {$i<=$PylonNP} {incr i 1} {
     		load $NodePylon($i) 0.0 0.0 [expr -$NodeMPylon($i)*$g] 0.0 0.0 0.0 
    	}
	for {set i 1} {$i<=$CrossBeamN} {incr i 1} {
		load $NodeCrBm($i) 0.0 0.0 [expr -$NodeMCrBm($i)*$g] 0.0 0.0 0.0
	}
	for {set i 1} {$i<=$OtherNP} {incr i 1} {
		load $NodeOther($i) 0.0 0.0 [expr -$NodeMOther($i)*$g] 0.0 0.0 0.0
	}
}
source recorder.tcl
source gravity.tcl
#source period.tcl
```
