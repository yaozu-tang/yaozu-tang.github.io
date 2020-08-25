---
title: Charge Density Difference Processing
date: 2020-08-25 20:51:57
tags: charge density
categories: research
description: A jupyter notebook script for processing the charge density difference in hybrid heterostructures
---

This jupyter(python3) script is used to plot the charge density difference in hybrid heterostructures, including the 2D contour figure(integration along one direction) and plane averaged figure(integration along two direction). The plane averaged figure could plot the charge density difference in difference parts of the system. This script is originally written by Dr. Ding. I modified this script in order to plot the 2D contour figure and separate the charge density difference in one system into a few parts according to the structural properties of the system.

```python
# %load /Users/tangyaozu/Science_Project/BS_Thesis_Project/1_TMD_hybrid_HS/Electronic Properties/Length_5-5/charge/plot_chg_upper.py
#!/usr/bin/python
# Dwj @  2018.10.07
# Tyz update @ 2020.7.20
%matplotlib inline
import numpy as np
import matplotlib as mpl
# mpl.use("PDF")
import matplotlib.pylab as pl
import matplotlib.pyplot as plt
from matplotlib.ticker import MaxNLocator
```

+ The following part is the function of reading the CHGCAR file. It loads and reads the CHARCAR file from the output of VASP static calculation. The data(charge number) in CHGCAR file is arranged by a certain order according to the real space mesh(NX, NY, NZ). This function serves to read the somehow complicated CHGCAR file and reconstruct it into a format that we can easily understand and put into use. We read the CHGCAR file from the first line to the last and put the information into several different array according to the physical meaning of the data. This function is rudimentary and can be put into any code related to charge density processing.

```python
#----------define the function of reading the CHGCAR file----------
def read_chgcar(filename):
    f = open(filename,'r')
    f.readline()
    lc = float(f.readline().split()[0])
    # lc is the scaling factor
    axis=np.zeros((3,3))
    axis[0]=[float(x) for x in f.readline().split()]
    axis[1]=[float(x) for x in f.readline().split()]
    axis[2]=[float(x) for x in f.readline().split()]
    # read the structure of the lattice
    atomlabels = f.readline().split()
    # read the atom label
    natoms = np.array([int(x) for x in f.readline().split()])
    # the number of total atoms
    nt=0
    for i in range(len(natoms)):
        nt += natoms[i];
    f.readline()
    pos_array=np.zeros((nt,3))
    for i in range(nt):
        pos_array[i] = np.array([float(x) for x in f.readline().split()])
    # read the position of each atom
    f.readline()
    NGXF,NGYF,NGZF = [int(x) for x in f.readline().split()]
    # read the real space mesh
    NGT = NGXF * NGYF * NGZF
    if NGT%5==0: nline = NGT//5;
    else : nline = NGT//5+1;
    # there are five charge number data each line in CHGCAR file
    chgcar = []
    for i in range(nline):
        line = [float(x) for x in f.readline().split()]
        for j in range(len(line)):
            chgcar.append(line[j])
    chgcar_array = np.array(chgcar).reshape((NGZF,NGYF,NGXF))
    # read charge number data and reshape it according to the real space mesh (3D matrice)
    NG_list = [NGXF,NGYF,NGZF]
    return [lc, axis, atomlabels, natoms, pos_array, NG_list, chgcar_array]
```

+ The following function serves to integrate the charge density difference data along y and z direction, which means only the x direction information will be remained. This is the so called plane averaged charge density difference. The fluctuation of charge density difference along a certain direction(x in my case) will be presented and the charge transfer in this direction can be examined.

```python
#----------define the function of charge intergration along two(yz) axis----------
def integral_yz(NGT,chg):
    ngx,ngy,ngz = NGT
    # load the FFT grid information
    tot=np.zeros((ngx))
    for i in range(ngx):
        for j in range(ngy):
            for k in range(ngz):
                tot[i] += chg[k][j][i]/ngx/ngy/ngz
    # integrate along y and z direction
    return tot
```

+ The following function serves to integrate the charge density difference data along only y direction, which means the transformation of charge density difference in a plane will be presented.

```python
#----------define the function of charge intergration along one(y) axis----------
def integral_y(NGT,chg):
    ngx,ngy,ngz = NGT
    # load the FFT grid information
    tot=np.zeros((ngx,ngz))
    for i in range(ngx):
        for j in range(ngz):
            for k in range(ngy):
                tot[i][j] += chg[j][k][i]/ngx/ngy/ngz
    # integrate along y direction
    return tot
```

+ The following part is the main function, which serves to call other functions previously defined and plot the contour and plane averaged charge density difference figure.

```python
#----------define the main function----------
#    file path
path_0 = '/Users/tangyaozu/Science_Project/BS_Thesis_Project/1_TMD_hybrid_HS/Electronic Properties/Length_5-5/charge/'
path= path_0 + 'CHGCAR_tot'
path_A1 = path_0 + 'CHGCAR_A1'
path_B1 = path_0 + 'CHGCAR_B1'
path_A2 = path_0 + 'CHGCAR_A2'
path_B2 = path_0 + 'CHGCAR_B2'
#    there are four different parts in my heterostructure system

#    read the CHGCAR file
[lc,axis,atomlabels,natoms,pos,NGT,chg] = read_chgcar(path)
[lc1,axis1,atomlabels1,natoms1,pos1,NGT1,chg1] = read_chgcar(path_A1)
[lc2,axis2,atomlabels2,natoms2,pos2,NGT2,chg2] = read_chgcar(path_B1)
[lc3,axis3,atomlabels3,natoms3,pos3,NGT3,chg3] = read_chgcar(path_A2)
[lc4,axis4,atomlabels4,natoms4,pos4,NGT4,chg4] = read_chgcar(path_B2)

#    calculate the x,y,z position of each grid point(it's actually a small box)
ngx,ngy,ngz = NGT
lc_x = axis[0][0] * lc
lc_y = axis[1][1] * lc
lc_z = axis[2][2] * lc
ddx = lc_x * 1.0/ngx
ddy = lc_y * 1.0/ngy
ddz = lc_z * 1.0/ngz
x = lc_x * 1.0/ngx * np.arange(ngx)
y = lc_y * 1.0/ngy * np.arange(ngy)
z = lc_z * 1.0/ngz * np.arange(ngz)

#    intergration along y and z axis
chgcar_x = integral_yz(NGT,chg)
chgcar_x1 = integral_yz(NGT1,chg1)
chgcar_x2 = integral_yz(NGT2,chg2)
chgcar_x3 = integral_yz(NGT3,chg3)
chgcar_x4 = integral_yz(NGT4,chg4)

#    calculate the charge difference along the x axis
diff_x_plot = (chgcar_x-chgcar_x1-chgcar_x2-chgcar_x3-chgcar_x4)/ddx
diff_x = (chgcar_x-chgcar_x1-chgcar_x2-chgcar_x3-chgcar_x4)
date = np.array([x,diff_x]).T

#    integration along y axis
chgcar_xz = integral_y(NGT,chg)
chgcar_xz1 = integral_y(NGT1,chg1)
chgcar_xz2 = integral_y(NGT2,chg2)
chgcar_xz3 = integral_y(NGT3,chg3)
chgcar_xz4 = integral_y(NGT4,chg4)

#    calculate the charge difference in xz plane
diff_xz_plot = (chgcar_xz-chgcar_xz1-chgcar_xz2-chgcar_xz3-chgcar_xz4)/ddx/ddz
diff_xz = (chgcar_xz-chgcar_xz1-chgcar_xz2-chgcar_xz3-chgcar_xz4)

#    reconstruct the charge difference array in xz plane
diff_xz_plot_new = np.concatenate((diff_xz_plot[int(15/ddx):560,:],diff_xz_plot[0:int(15/ddx),:]))
#    reconstruct the charge difference array to put the two different junction in the center of x axis.

#    plot the charge difference contour figure in xz plane
X,Z = np.meshgrid(np.array(x),np.array(z))
plt.figure(num = 2,figsize=(20,5))
levels = MaxNLocator(nbins=200).tick_values(np.min(diff_xz_plot_new),np.max(diff_xz_plot_new))
cm = plt.cm.get_cmap('rainbow')
C = plt.contourf(X, Z, diff_xz_plot_new.T, levels=levels,cmap=cm)
font = {'family' : 'serif',
        'color'  : 'black',
        'weight' : 'normal',
        'size'   : 16,
        }
cb = plt.colorbar(C)
cb.set_label('charge transfer e/Å^2',fontdict=font)
cb.ax.tick_params(labelsize=12)
#plt.xlim((20,35))
plt.xticks(fontsize=12)
plt.yticks(fontsize=12)
plt.xlabel('a-axis position (Å)',fontdict=font)
plt.ylabel('c-axis position (Å)',fontdict=font)
plt.ylim((6,20))
plt.savefig('./charge_xz.png')
plt.show()

#    calculate the charge difference of up and down sheets along the x direction
diff_xz_down = diff_xz_plot[:,0:128]
diff_xz_up = diff_xz_plot[:,128:256]
diff_x_down = diff_xz_down.sum(axis=1)*ddz
diff_x_up = diff_xz_up.sum(axis=1)*ddz
#    there are four parts in my system, two laterally connected are placed in the upper position,
#and the other two laterally connected are placed in the lower part

#    reconstruct the charge difference array along the x direction
diff_x_down_new = np.append(diff_x_down[int(15/ddx):int(40/ddx)], diff_x_down[int(40/ddx):560])
diff_x_down_new = np.append(diff_x_down_new, diff_x_down[0:int(15/ddx)])
diff_x_up_new = np.append(diff_x_up[int(15/ddx):int(40/ddx)], diff_x_up[int(40/ddx):560])
diff_x_up_new = np.append(diff_x_up_new,diff_x_up[0:int(15/ddx)])
#    reconstruct the charge difference array to put the two different junction in the center of x axis.

#    plot the charge difference along the x direction in half of z-scope
fig4 = plt.figure(num=4,figsize=(8,6))
down_plot_new = fig4.add_axes([0.1,0.1,0.8,0.2])
plt.xlabel('a-axis position(Å)',fontdict=font)
up_plot_new = fig4.add_axes([0.1,0.3,0.8,0.2],xticklabels=[],xticks=[])
junction_1 = fig4.add_axes([0.1,0.5,0.4,0.4],xticklabels=[],xticks=[],xlim=(5,20))
junction_2 = fig4.add_axes([0.5,0.5,0.4,0.4],xticklabels=[],xticks=[],xlim=(33,48),yticklabels=[],yticks=[])
l1, = down_plot_new.plot(x,diff_x_down_new,'b-',label='lower plane')
l2, = up_plot_new.plot(x,diff_x_up_new,'r-',label='upper plane')
l3, = junction_1.plot(x,diff_x_up_new, color='red', linestyle='-')
l4, = junction_1.plot(x,diff_x_down_new,color='blue', linestyle='-')
l5, = junction_2.plot(x,diff_x_up_new, color='red', linestyle='-')
l6, = junction_2.plot(x,diff_x_down_new, color='blue', linestyle='-')
#fig4.text(0.5,-0.15,'a-axis position (Å)',ha='center',fontdict=font)
#fig4.text(-0.15,0.5,'charge transfer (e/Å)',va='center',rotation='vertical',fontdict=font)
fig4.legend(handles=[l1,l2],loc=1)
plt.savefig('./charge_x.png')
plt.show()
```

This script is still crude, so it's not posted on my Github. There's going to be continuous modification and update on his script, I would appreciate any suggestion!
