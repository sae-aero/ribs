# ribs
james and hannah's rib truss issues
#H. Green
#Wing Rib Trus Code

#This code pulls from an excel file with airfoil data
#In order to use, open the excel file i have specified
#further details/instructions are there. After changing the data,
#close the excel file, as the code will not run with the file open.
#Use a higher resolution excel file to get most accurate results
import copy
from math import *
import numpy as np
from openpyxl import *
import matplotlib.pyplot as plt


#Step 1: Import From Excel File

wb = load_workbook("0012.xlsx")
sheet = wb.active

cols = sheet.columns

values = []
for col in cols:
    for cell in col:
        if cell.value != None:
            values.append(cell.value)

#break into top/bottom/x/y values

x = len(values)
xvals = values[0:int(x/2)]
yvals = values[int(x/2):len(values)+1]
x = int(len(xvals)/2)
uwx = xvals[0:x] #Upper wing x values
lwx = xvals[x:int(len(xvals))+1] #lower wing x values
uwy = yvals[0:x] #upper wing y values
lwy = yvals[x:int(len(xvals))+1] #lower wing y values

#Set spar Positions
fwdspar = 25 #Percent Chord from the LE
aftspar = 75  #Percent Chord from the LE

#Split Lists of values based on the spar locations
splitptfwd = int(fwdspar*len(uwx)/100)
splitptaft = int(aftspar*len(uwx)/100)

centralx = uwx[splitptfwd:splitptaft] #xrange in between spars
uppery = uwy[splitptfwd:splitptaft] # Y values of top surface of airfoil
lowery = lwy[splitptfwd:splitptaft] # Y values of lower surface of airfoil

## STEP 2: Determine nodes for truss sections

#Choose amount of sections- see whitepaper for definition of section
#eventually hopefully the code will iterate through to really optimize this
#without having to do so manually.

numsections = 3 #Change manually
x = len(centralx)/numsections

nodesx = []
nodesyupper = []
nodesylower = []
#divide sections & pick nodes

i = 0
while i <= len(centralx):
    nodesx.append(centralx[i])
    nodesylower.append(lowery[i])
    nodesyupper.append(uppery[i])

    i += int(len(centralx)/(numsections+1))

# print(nodesylower)
# print(nodesyupper)

i = 0
while i < len(nodesx):
    plt.plot(nodesx[i], nodesyupper[i], 'o', color='b')
    plt.plot(nodesx[i], nodesylower[i], 'o', color='g')
    i += 1

plt.show()

#STEP 3: Direct Stiffness amtrix

#Define Elements- length and angle

i = 0
hormemtops = []
hormembottoms = []
diagmems = []
vertmems = []


print(nodesx)
print(nodesyupper)
print(nodesylower)

# Define Lengths

#Vertical Members
v = []
i =0
while i < len(nodesylower):
    L = round(nodesyupper[i]-nodesylower[i],3)
    v.append(L)
    i += 1
print("vertical",v)

# "Horizontial" members
hb = [] #horizontial bottom members
ht = [] #horizontial top members
i = 1
while i < len(nodesylower):
    Lb = round(sqrt(((nodesx[i]-nodesx[i-1])**2+(nodesylower[i]-nodesylower[i-1])**2)),3)
    Lt = round(sqrt(((nodesx[i]-nodesx[i-1])**2+(nodesyupper[i]-nodesyupper[i-1])**2)),3)
    hb.append(Lb)
    ht.append(Lt)
    i += 1

print("H bottom", hb)
print("H top", ht)

#Define Diagonal members

d = []
i = 1
while i < len(nodesylower):
    if i // 2 == 0:
        L = round(sqrt(((nodesx[i]-nodesx[i-1])**2+(nodesylower[i]-nodesyupper[i-1])**2)),3)
        d.append(L)
        i += 1
    else:
        L = round(sqrt(((nodesx[i] - nodesx[i - 1]) ** 2 + (nodesyupper[i] - nodesylower[i - 1]) ** 2)), 3)
        d.append(L)
        i += 1
print('diagonal', d)

##Define angles
va = [] #vertical angles
i = 0
while i < len(nodesylower):
    theta = round(degrees(acos((nodesx[i]-nodesx[i])/(nodesyupper[i]-nodesylower[i]))),3)
    va.append(theta)
    i += 1
print(va)

hta = [] #"horizontial" toop angles
i = 1
while i < len(nodesylower):
    theta = round(degrees(atan((nodesyupper[i]-nodesyupper[i-1])/(nodesx[i]-nodesx[i-1]))),3)
    hta.append(theta)
    i += 1
print(hta)

hba = [] #horizontial bottom angles
i = 1
while i < len(nodesylower):
    theta = round(degrees(atan((nodesylower[i]-nodesylower[i-1])/(nodesx[i]-nodesx[i-1]))),3)
    hba.append(theta)
    i += 1
print(hba)

da = []

i = 1
while i < len(nodesylower):
    if i % 2 == 0:
        theta = round(degrees(atan((nodesylower[i]-nodesyupper[i])/(nodesx[i]-nodesx[i-1]))), 3)
        print(i)
        da.append(theta)
        i += 1
    elif i % 2 != 0:
        theta = round(degrees(atan((nodesyupper[i] - nodesylower[i - 1]) / (nodesx[i] - nodesx[i - 1]))), 3)
        da.append(theta)
        i += 1
print(da)

# Automates Connectivity

memberslen = []
i=0
while i < len(v)-1:
    memberslen.append(v[i])
    i += 1
i=0
while i < len(v)-1:
    memberslen.append(ht[i])
    i += 1
i=0
while i < len(v)-1:
    memberslen.append(hb[i])
    i += 1
i=0
while i < len(v)-1:
    memberslen.append(d[i])
    i += 1
print("member lengths- in order", memberslen) # "names" the members (1-##)

#Match Connectivity to Lengths and Angles

#Vertical Lengths to Connevtivity



