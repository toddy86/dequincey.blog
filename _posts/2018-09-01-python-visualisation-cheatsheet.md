---
layout: post
title: Python Visualisation Cheat Sheet
categories: projects
---

A cheat sheet of core data visualisation techniques using matplotlib and seaborn.

<!-- more -->
## Project goal

For one reason or another, the matplotlib syntax constantly escapes me. So, I have created this notebook to slowly work through the ins and outs of matplotlib and seaborn in hope to 1) cement the knowledge in my head and/or 2) create a personal cheat sheet which I can refer to when I undoubtably forget the syntax again.

Whilst there are a range of matplotlib and seaborn resources and cheat sheets already available, the best way for me to learn is to write, explain and teach. Thus, my recreation of the matplotlib basics wheel here.

## Resources

The following is a list of useful matplotlib and seaborn resources:
<ul>
    <li><a href="https://realpython.com/python-matplotlib-guide/">Real Python: Python Plotting with Matplotlib Guide</a></li>
    <li><a href="https://seaborn.pydata.org/tutorial.html">Seaborn Tutorial</a></li>
    <li><a href="https://github.com/jakevdp/PythonDataScienceHandbook">Python Data Science Handbook</a></li>
</ul>



## Import dependencies


```python
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
import pandas as pd
import numpy as np

import warnings 
warnings.filterwarnings('ignore')
```

## Load dummy dataset
The seaborn package comes with some pre-defined datasets to make our life easy, so we will load the titanic dataset


```python
# Display the available seaborn datasets (requires an internet connection)
sns.get_dataset_names()
```




    ['anscombe',
     'attention',
     'brain_networks',
     'car_crashes',
     'diamonds',
     'dots',
     'exercise',
     'flights',
     'fmri',
     'gammas',
     'iris',
     'mpg',
     'planets',
     'tips',
     'titanic']




```python
# Load tips dataset
df = sns.load_dataset('tips')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>total_bill</th>
      <th>tip</th>
      <th>sex</th>
      <th>smoker</th>
      <th>day</th>
      <th>time</th>
      <th>size</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>16.99</td>
      <td>1.01</td>
      <td>Female</td>
      <td>No</td>
      <td>Sun</td>
      <td>Dinner</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10.34</td>
      <td>1.66</td>
      <td>Male</td>
      <td>No</td>
      <td>Sun</td>
      <td>Dinner</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>21.01</td>
      <td>3.50</td>
      <td>Male</td>
      <td>No</td>
      <td>Sun</td>
      <td>Dinner</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>23.68</td>
      <td>3.31</td>
      <td>Male</td>
      <td>No</td>
      <td>Sun</td>
      <td>Dinner</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>24.59</td>
      <td>3.61</td>
      <td>Female</td>
      <td>No</td>
      <td>Sun</td>
      <td>Dinner</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



## Basic plots

### Setting a plot style


```python
plt.style.use('seaborn-white')
```

#### Creating a single plot

There are two basic methods to plotting with matplotlib:
1. Using the matlab style syntax
2. Using object-oriented style syntax

#### Matlab style


```python
# Group the number of tips received by day and split out into an x and y varible
grouped_by_day = df.groupby('day').sum()
days = grouped_by_day.index
tip_amt = grouped_by_day['tip']
bill_amt = grouped_by_day['total_bill'] # Will be used later
```


```python
# Create a figure
plt.figure()

# Plot the data
plt.bar(days, tip_amt);
```


![png](/assets/images/data-vis/output_16_0.png)


#### Object-oriented style
NB: Could plot directly from DF - this is covered later


```python
# Create a stateless (OO) graph using the plt.subplots() method
fig, ax = plt.subplots(figsize=(8,4))

# Use ax.type_of_plot and pass in x and y
ax.bar(days, tip_amt);
```


![png](/assets/images/data-vis/output_18_0.png)


For simplicity, the rest of this notebook uses the object-orientated style of plotting, as this is generally more useful as plotting needs get more complicated.

#### Titles


```python
# Add a title 
ax.set_title("Total Tips by Day of the Week")

# Display the plot
fig
```




![png](/assets/images/data-vis/output_21_0.png)



#### Axis Labels


```python
# Add axis labels
ax.set_xlabel('Day of the Week')
ax.set_ylabel('Total Tips Received ($)')

# Display the plot
fig
```




![png](/assets/images/data-vis/output_23_0.png)



#### Legends


```python
#Create a legend
ax.legend(['Tip'])

# Display plot
fig
```




![png](/assets/images/data-vis/output_25_0.png)



### Annotating plots with text
Use ax.text(x_pos, y_pos, 'text') on object-oriented plots


```python
# Annotate Friday as low
ax.text(1, 60, "Low tips on Friday. Why?", ha='center')
fig
```




![png](/assets/images/data-vis/output_27_0.png)



### Annotating plots with arrows and text
Use ax.annotate('text', xy=(coord_of_arrow), xytext=(coord_of_text)) on object-oriented plots


```python
# Annotate Thursday with an arrow
ax.annotate('Seem to do well on Thursday', 
            xy=(0, 170), 
            xytext=(-1.5, 240), 
            arrowprops=dict(arrowstyle='->'), 
            ha='center')
fig
```




![png](/assets/images/data-vis/output_29_0.png)



#### Plotting multiple elements


```python
# Create length of the plot and set the width of the bars
ind = np.arange(len(days))
width = 0.35

# Create a figure
fig2, ax2 = plt.subplots()

# Use ax.type_of_plot and pass in x and y
ax2.bar(ind, tip_amt, width=width);
ax2.bar(ind + width ,bill_amt, width=width);

# Set the xticks width and set xtickslabels to days
ax2.set_xticks(ind + width / 2)
ax2.set_xticklabels(days)

#Create a legend
ax2.legend(['Tip', 'Total Bill']);
```


![png](/assets/images/data-vis/output_31_0.png)


## Multiple plots

### Subplots


```python
# Create 2 subplots, which share the same x axis labels
fig, ax = plt.subplots(2, sharex=True)

# Plot axis 0 and 1 as tip_amt and bill_amt
ax[0].bar(days, tip_amt)
ax[1].bar(days, bill_amt, color='orange')
```




    <BarContainer object of 4 artists>




![png](/assets/images/data-vis/output_34_1.png)


### Creating a grid


```python
# Create a grid of 2 rows x 3 cols
# sharex and sharey removes inner x and y labelling of each plot
fig, ax = plt.subplots(2, 3, sharex='col', sharey='row')

# plot to each of the subplots (could have done with a for loop, but wanted to be explicit)
ax[0,0].text(0.5, 0.5, str((0, 0)), fontsize=18, ha='center')
ax[0,1].text(0.5, 0.5, str((0, 1)), fontsize=18, ha='center')
ax[0,2].text(0.5, 0.5, str((0, 2)), fontsize=18, ha='center')
ax[1,0].text(0.5, 0.5, str((1, 0)), fontsize=18, ha='center')
ax[1,1].text(0.5, 0.5, str((1, 1)), fontsize=18, ha='center')
ax[1,2].text(0.5, 0.5, str((1, 2)), fontsize=18, ha='center');
```


![png](/assets/images/data-vis/output_36_0.png)


###  GridSpec for custom layouts
Below example adapted from the <a href="https://github.com/jakevdp/PythonDataScienceHandbook"> Data Science Handbook</a>.


```python
# Create x and y variables
x = df['total_bill']
y = df['tip']

# Create a figure which is 6x6 in size 
fig = plt.figure(figsize=(6, 6))

# Create a GridSpec 
grid = plt.GridSpec(4, 4, hspace=0.2, wspace=0.4)

# Create multiple plots add arrange on the 6x6 grid
main_scat = fig.add_subplot(grid[:-1, 1:])
y_hist = fig.add_subplot(grid[:-1, 0], xticklabels = [], sharey=main_scat)
x_hist = fig.add_subplot(grid[-1, 1:], yticklabels = [], sharex=main_scat)


main_scat.plot(x, y, 'ok', markersize=4)

x_hist.hist(x, orientation='vertical')
x_hist.invert_yaxis()

y_hist.hist(y, orientation='horizontal')
y_hist.invert_xaxis()
```


![png](/assets/images/data-vis/output_38_0.png)


## Plotting with pandas
Rather than splitting out variables from a pandas dataframe into separate variables for plotting, we can directly plot from the dataframe using the pd.plot() method. 


```python
# Plotting a scatter plot directly from a pandas DF
df.plot.scatter('total_bill', 'tip');
```


![png](/assets/images/data-vis/output_40_0.png)


#### Combing with pandas groupby


```python
# Plot a Combine with pd.groupby to display the sum of tips and bill per day
df.groupby('day')[['tip', 'total_bill']].sum().plot(kind='bar');
```


![png](/assets/images/data-vis/output_42_0.png)



```python
# Plot separate subplots - pass subplots=True arg
df.groupby('day')[['tip', 'total_bill']].sum().plot(kind='bar', subplots=True);
```


![png](/assets/images/data-vis/output_43_0.png)


## Seaborn
Seaborn is a package built on top of matplotlib which extends its functionality (e.g. adds violin plots) and provides some aesthetic improvements to the base matplotlib style of plotting.

Many of the below examples are adapted from the <a href"https://seaborn.pydata.org/tutorial.html">seaborn docs</a>

### Categorical plots
Can either use the sns.catplot() method and pass in the kind arg, or directly create the required plot by calling the appropriate sns plot (e.g. sns.swarmplot() )

#### Basic plot


```python
# Create a swarmplot
sns.catplot(x = 'day', y = 'tip', kind='swarm', data = df);
```


![png](/assets/images/data-vis/output_47_0.png)


#### Adding hue


```python
# Display points split by sex 
sns.catplot(x = 'day', y = 'tip', kind='swarm', hue='sex', data = df);
```


![png](/assets/images/data-vis/output_49_0.png)


#### Conditional display with queries


```python
# Only display tips greater than $2
sns.catplot(x = 'day', y = 'tip', hue = 'sex', data = df.query('tip > 2'));
```


![png](/assets/images/data-vis/output_51_0.png)


#### Ordering


```python
# Show tips by male or female
sns.catplot(x = 'sex', y = 'tip', order=['Male', 'Female'], data = df);
```


![png](/assets/images/data-vis/output_53_0.png)


#### Displaying multiple plots


```python
sns.catplot(x='day', y='tip', hue='sex', col='time', kind='bar', data=df);
```


![png](/assets/images/data-vis/output_55_0.png)


### Continious plots
Similar to categorical plots, we can either use the relplot() "parent" method and pass in the kind arg, or directly choose the type of plot by calling it (e.g. sns.scatterplot() ).

#### Basic plot


```python
# Scatter plot of total_bill vs tips
sns.relplot(x='total_bill', y='tip', data=df);
```


![png](/assets/images/data-vis/output_58_0.png)


#### Adding hue and styles


```python
# Add hue of sex and style of whether they are a smoker or not
sns.relplot(x='total_bill', y='tip', hue='sex', style='smoker', data=df);
```


![png](/assets/images/data-vis/output_60_0.png)


#### Multiple plots
Similar to catplot(), we can display multiple plots for continious variables.


```python
# Display total_bill vs tip, split by sex
sns.relplot(x='total_bill', y='tip', col='sex', data=df);
```


![png](/assets/images/data-vis/output_62_0.png)



```python
# Plot mutliple plots by the size of the group
sns.relplot(x='total_bill', y='tip', col='size', col_wrap=3, data=df);
```


![png](/assets/images/data-vis/output_63_0.png)


## Types of plots

### Matplotlib plots


```python
# Scatter
df.plot.scatter('total_bill', 'tip');
```


![png](/assets/images/data-vis/output_66_0.png)



```python
# Bar
df.groupby('day')[['tip']].sum().plot.bar();
```


![png](/assets/images/data-vis/output_67_0.png)



```python
# Horizontal Bar
df.groupby('day')[['tip']].sum().plot.barh();
```


![png](/assets/images/data-vis/output_68_0.png)



```python
# Pie
df.groupby('day')[['tip']].sum().plot.pie(subplots=True);
```


![png](/assets/images/data-vis/output_69_0.png)



```python
# Box plot
df[['size', 'tip']].plot.box();
```


![png](/assets/images/data-vis/output_70_0.png)



```python
# Histogram
df['tip'].plot.hist();
```


![png](/assets/images/data-vis/output_71_0.png)


Other matplotlib plots which aren't shown above include:
- area
- density
- hexbin
- kernal density estimation
- line

### Seaborn plots
Seaborn includes all of the same plots as matplotlib, with some additional plots.


```python
# Strip plot
sns.stripplot(x = 'day', y = 'tip', data = df, jitter=True);
```


![png](/assets/images/data-vis/output_74_0.png)



```python
# Swarm plot
sns.swarmplot(x = 'day', y = 'tip', data = df, hue='sex');
```


![png](/assets/images/data-vis/output_75_0.png)



```python
# Violin plots
sns.violinplot(x = 'tip', y = 'day', hue='time', data = df);
```


![png](/assets/images/data-vis/output_76_0.png)



```python
# Pairplot
sns.pairplot(df);
```


![png](/assets/images/data-vis/output_77_0.png)



```python
# Jointplot
sns.jointplot(x = 'total_bill', y = 'tip', data = df);
```


![png](/assets/images/data-vis/output_78_0.png)

