---
layout: single
title: Matplotlib Note(Python)
categories: Coding
toc: true
toc_sticky: true
toc_label: Contents
author_profile: false
---

```python
import numpy as np
import matplotlib.pyplot as plt

```

# Example plot 1 - pyplot API Style
This is a simple plotting example



```python
y = np.array([10,20,15])
plt.plot(y)
```




    [<matplotlib.lines.Line2D at 0x7ff06dc00490>]




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_2_1.png)
    

```python
x = np.array([5.,10.,15.])
y = np.array([10.,20.,15.])
```


```python
plt.plot(x,y)
```




    [<matplotlib.lines.Line2D at 0x7ff06df6a290>]




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_4_1.png)
    



```python
plt.plot(x,y, 'r')
```




    [<matplotlib.lines.Line2D at 0x7ff06d609d10>]




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_5_1.png)
    



```python
plt.plot(x,y,'b--')
```




    [<matplotlib.lines.Line2D at 0x7ff06d6149d0>]




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_6_1.png)
    



```python
plt.plot(x,y, 'bo')
```




    [<matplotlib.lines.Line2D at 0x7ff06d574a50>]




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_7_1.png)
    



```python
plt.plot(x,y, 'b-o', label='Data 1')
plt.ylabel('Y axis')
plt.xlabel('X axis')
plt.legend()
```




    <matplotlib.legend.Legend at 0x7ff06d4a9250>




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_8_1.png)
    


# Plot Example 2 - Object-Oriented Style



```python
x
```




    array([ 5., 10., 15.])




```python
y
```




    array([10., 20., 15.])




```python
hf = plt.figure()
ha = hf.add_axes([0,0,1,1])
ha.plot(x,y)
```




    [<matplotlib.lines.Line2D at 0x7ff06d2ef990>]




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_12_1.png)
    



```python
fig, ax = plt.subplots()
line, =ax.plot(x,y)
```


    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_13_0.png)
    



```python
i = 1.1
x_new = np.array([10,11,12])
y_new = np.array([10,10,10])
line.set_data(x_new,i*y_new)
fig
```




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_14_0.png)
    



Multiple Line Graphs = Pyplot style


```python
x = np.linspace(0,2,100)
y1 = 0.5*x
y2 = 0.5*x**2
y3 = 0.5*x**3
```


```python
plt.plot(x,y1,label='Linear')
plt.plot(x,y2,label='Quadratic')
plt.plot(x,y3,label='Cubic')
plt.legend()
plt.ylabel('Y')
plt.xlabel('X')
plt.title('Graphs')
```




    Text(0.5, 1.0, 'Graphs')




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_17_1.png)
    


Multiple Line Graphs - OOP Style


```python
hf, ha = plt.subplots()
hp1, = ha.plot(x,y1)
hp2, = ha.plot(x,y2)
hp3, = ha.plot(x,y3)
ha.set_ylabel('Y')
ha.set_xlabel('X')
ha.set_title('Graphs')
hp1.set_label('Linear')
hp2.set_label('Quadratic')
hp3.set_label('Cubic')
ha.legend()
```




    <matplotlib.legend.Legend at 0x7ff06d155d90>




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_19_1.png)
    



```python
hp1.set_color('k')
hp2.set_color('b')
hp3.set_color('r')
ha.legend()
ha.grid()
hf
```




    
![png]({{site.url}}/assets/images/matplotlib_Practice_files/matplotlib_Practice_20_0.png)
    
