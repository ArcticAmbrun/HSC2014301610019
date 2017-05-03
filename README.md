# HSC2014301610019
marine charting
#student came from WHU,somework to finish HSC
#use python





# -*- coding: utf-8 -*-
"""
Created on Wed Apr 05 22:10:20 2017

@author: Rain
"""
def SortList(X1,X2):
    X3=0
    X3=X1
    X1=X2
    X2=X3
    return X1,X2

def RayTracingg0(SVP_H,SVP_C,Theta,T):
    DeltaHI=[0]*len(SVP_H)
    ThetaI=[0]*len(SVP_H)
    YI=[0]*len(SVP_H)
    TI=[0]*len(SVP_H)
    Y=0
    H=0
    DeltaTime=0
    timerecent=0#定义一个临时时间，用于和time进行对比
    time=T/2#time为传播一半的时间
    for q in range(0,len(SVP_H)-1):
        if(q==0):
            DeltaHI[q]=SVP_H[q]            
            ThetaI[q]=Theta
        else:
            DeltaHI[q]=SVP_H[q]-SVP_H[q-1]
            ThetaI[q]=round(math.asin(math.sin(ThetaI[q-1])*SVP_C[q]/SVP_C[q-1]),10)
    #为每一阶段的高差,入射角赋值
    for w in range(0,len(SVP_H)-1):
        YI[w]=round(DeltaHI[w]*math.tan(ThetaI[w]),10)
        TI[w]=round(YI[w]/(SVP_C[w]*math.sin(ThetaI[w])),10)
        timerecent+=TI[w]
        Y+=YI[w]
        H+=DeltaHI[w]
        
        if(timerecent>time and w<=len(SVP_H)-1):#如果段落时间大于传播时间并且在序列内，说明在该水层停止，测深在此区域内
            timerecent-=TI[w]
            Y-=YI[w]        #首先做一步减法
            H-=DeltaHI[w]
            DeltaTime=time-timerecent
            Y+=round(math.sin(ThetaI[w])*SVP_C[w]*DeltaTime,10)
            H+=round(math.cos(ThetaI[w])*SVP_C[w]*DeltaTime,10)            
            break
        else:
            if(timerecent<time and w==len(SVP_H)-1):#所有段落均完成，说明有底部的SVP
                DeltaTime=time-timerecent
                Y+=round(math.sin(ThetaI[w])*SVP_C[w]*DeltaTime,10)
                H+=round(math.sin(ThetaI[w])*SVP_C[w]*DeltaTime,10)

                break
    return Y,H

def RayTracinggnot0(SVP_H,SVP_C,Theta,T):#层内常梯度
    return 0

def Convert(Rad,X,Y):#Rad为航向，单位为弧度
    X1=X*math.cos(Rad)+Y*math.sin(Rad)
    Y1=Y*math.cos(Rad)-X*math.sin(Rad)
    return X1,Y1

'''
'''

import math
import xlrd#打开xls文件
import string
import copy
import numpy

#绘图包
import matplotlib.pyplot as plt
from matplotlib import cm
from mpl_toolkits.mplot3d import Axes3D#不晓得怎么用，好像不是很好用

book = xlrd.open_workbook(r"C:\Users\Rain\hsc2014301610019\RawData.xlsx")
#通过名称获取第一个工作表
table=book.sheet_by_name(u"Sheet1")
Ping=table.col_values(0)
Beam=table.col_values(1)
X=table.col_values(2)
Y=table.col_values(3)
H=table.col_values(4)
Theta=table.col_values(5)
S=table.col_values(6)
Rad=table.col_values(7)

SVP=open("C:\Users\Rain\hsc2014301610019\SVP.txt","r")
content=SVP.readlines()
#前三行没有意义，有意义就是好好活，好好活就是有意义
Time=len(content)-3
SVP1=[0]*Time#对应深度depth
SvpDepth=copy.deepcopy(SVP1)
SVP2=[0]*Time#声速C
SvpC=copy.deepcopy(SVP2)
for q in range(0,Time):
    SVP0=content[q+3].split()
    SVP1[q]=SVP0[0]
    SVP2[q]=SVP0[1]
for w in range(0,Time):
    SvpDepth[w]=string.atof(SVP1[w])
    SvpC[w]=string.atof(SVP2[w])
MaxDepth=max(SvpDepth)
MaxC=max(SvpC)
MaxDepthPosition=SvpDepth.index(MaxDepth)
DAB=[0]*Time
CAB=[0]*Time
DAA=[0]*Time
CAA=[0]*Time
#D->Depth C->C ||  A->average || B->before A->after  

#序列应该有纠正的能力
Depth1=0
Depth2=0
C1=0
C2=0
#目的为比较点
for e in range(0,MaxDepthPosition):
    Depth1=SvpDepth[e]
    Depth2=SvpDepth[Time-1-e]
    C1=SvpC[e]  
    C2=SvpC[Time-1-e]
    if(Depth1>=Depth2):
        Depth1,Depth2=SortList(Depth1,Depth2)
        C1,C2=SortList(C1,C2)
    DAB[2*e]=Depth1
    DAB[2*e+1]=Depth2
    CAB[2*e]=C1
    CAB[2*e+1]=C2
#进行数组重排列
for r in range(0,Time-1):
    if(DAB[r]>DAB[r+1]):
        DAB[r],DAB[r+1]=SortList(DAB[r],DAB[r+1])
        CAB[r],CAB[r+1]=SortList(CAB[r],CAB[r+1])
print "排序完成"
#排序完成
tolerate=0.3#容许值在0.3以内
a=0
t=0
while t<Time-1:
    if(numpy.abs(DAB[t]-DAB[t+1])<= tolerate):#如果差值在限定范围内
        DAA[a]=round((DAB[t]+DAB[t+1])/2,4)
        CAA[a]=round((CAB[t]+CAB[t+1])/2,4)
        t=t+1
    else:
        DAA[a]=DAB[t]
        CAA[a]=CAB[t]
    a=a+1
    t=t+1
    if(DAA[a]==0):
        DAA.remove(0)
        CAA.remove(0)
A=len(DAA)
B=len(CAA)
#根据θ和已知的SVP，建立H-T关系，得到偏移量Y
print "第一阶段完成"
YShift=[0]*(len(Y)-1)
Depth=[0]*(len(Y)-1)
X0=[0]*(len(Y)-1)
Y0=[0]*(len(Y)-1)
#X,Y基础数据都是3位

for e in range(0,len(Y)-1):
    YShift[e],Depth[e]=RayTracingg0(DAA,CAA,Theta[e+1],S[e+1])
#计算在船体坐标系下的坐标，得到（X1,Y1）
print "done"
X1=[0]*len(Y0)
Y1=[0]*len(Y0)
H1=[0]*len(Y0)
for r in range(0,len(Y0)):
    X0[r],Y0[r]=Convert(Rad[r+1],0,YShift[r])
    X1[r]=X[r+1]+X0[r]
    Y1[r]=Y[r+1]+Y0[r]
    H1[r]=H[r+1]-Depth[r]
#根据读取的船体坐标，转换坐标为大地坐标系(X1,Y1,H1)
fileoutput = open('output.txt','w')
fileoutput.write('X'+'\t'+'\t'+'Y'+'\t'+'\t'+'H'+'\n')
for t in range(0,len(Y0)):
    fileoutput.write(str(round(X1[t],6))+'\t')
    fileoutput.write(str(round(Y1[t],6))+'\t')
    fileoutput.write(str(round(H1[t],6))+'\n')
fileoutput.close()
print "done"
#plot画图

# View it.
#s = mlab.mesh(X1,Y1,H1, representation="wireframe", line_width=1.0 )
#mlab.show()
fig = plt.figure(1)
ax = fig.gca(projection='3d')
ax.plot_trisurf(X1,Y1,H1, cmap=cm.jet, linewidth=0.01)
plt.show()




























