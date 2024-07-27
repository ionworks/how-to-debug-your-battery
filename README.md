# How to debug your battery
### By ![Tom Tranter](https://www.linkedin.com/in/tom-tranter-355a1033/), Co-founder & CTO at ![Ionworks](ionworks.com)


Picture the scene, you're an engineer at an electric vehicle company and your boss comes to you and says: *"Jeremy we've got a problem!"* (your name is Jeremy by the way) *"It's these damn batteries, there's just too many of them in the car and it's weighing us down but we really want our customers to be able to drive 400 miles without having to stop and charge!"* - What do you do? How do you debug your battery? It's a black box of magic right?!? Wrong. Use simulation.

## The **"and"** problem


It seems like there's no good solution in the market today. Batteries are either designed to be high energy **or** high power, not high energy **and** power. If we stick a bunch of high energy batteries in the vehicle we might get more range but then when we try to accelerate they overheat because the rated power is too low and the losses in the system are too high. This causes us to over-engineer the cooling system (adding weight and cost) and we also find that batteries operating at higher temperatures lose capacity faster. Also there are other constraints on the battery design, such as safety, lifetime, weight and last but by no means least cost.

Below is a spider-plot of the main metrics that battery engineers tend to care about when designing or selecting cells which often compete with one another. [1]

![](figures/spider.png)

A good intro into cell design can be found at ![batterydesign.net](https://www.batterydesign.net/power-versus-energy-cells/) [2]. Below is an image taken from that site of the difference between high power and high energy. Thinner electrodes tend to be better for drawing larger currents because there is less material in the way of ion transport but they have more inactive material (i.e. current colllectors - the yellow bits) per unit volume making them less energy dense.

![](figures/power-vs-energy-cell-b.webp)

This problem is even bigger for your buddy designing batteries for electric aircraft as the peak power needed for take-off and landing is much higher (10 x) than the power needed for cruising. So you have ot prioritize power and sacrifice range - aircraft also find it really hard to top-up charge mid flight. For a great perspective on the prospects of electric flight I recommend ![this aritcle](https://www.nature.com/articles/s41586-021-04139-1).

Luckily you know an engineer at your battery supplier and can call them to discuss the problem.

## The curse of dimensionality

The battery supplier is well aware of the difficulties in designing batteries and explains they have been spending years on the problem, testing different designs trying to optimize for different use cases. It takes a week to build and fully test one single design in the lab with all the characterization experiments they need to run and up to 3 months for cells they select for accelerated ageing tests. This is where the cell is cycled really hard constantly for at least a month and the data collected is then extrapolated to a real-world driving scenario (car mostly parked on your driveway), but going from one scenario to another isn't so straight-forward as the ageing effects are non-linear. A rough estimate for how much it costs to build and test a new battery design on a single cell (one data point) is around $1000 - $1500.

That might not sound too much but now consider how many different design decisions go into making one battery. You have to choose materials:
- Anode
- Cathode
- Electrolyte
- Separator

Each solid material can be tuned by:
- Thickness
- Porosity (amount of space for solid active material or for liquid ion transporting material)
- Particle size
- Composition if a mixture of materials

Then you have geometric factors like
- Form factor - cylindrical, prismatic, pouch
- Size i.e. 18650
- Number of tabs
- Current collector thickness
- Safety features like venting cap and central mandrel.

Each electrolyte composition can be altered too and this is often where the secret sauce actually is (but maybe more on that another day).

Let's say conservatively there are 20 things you might want to change. If you collect 3 different data points changing each thing one at a time (original design, some number higher, some number lower) whilst keeping everything else constant (usually a good scientific approach) that's 3<sup>20</sup> possible combinations of changes!

That's 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3 x 3

= 3,486,784,401

![](figures/counting.gif)

The short script below illustrates the curse for a 3 variable problem. It's hard to visualize a 20 dimensional parameter space but you get the drift.

```
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import numpy as np

def plot_points(x, y, z):
    # Create a 3D plot
    fig = plt.figure(figsize=(5, 5))
    ax = fig.add_subplot(111, projection='3d')

    # Plot the points
    ax.scatter(x.flatten(), y.flatten(), z.flatten(), s=10)

    # Set titles and labels
    ax.set_title(f'{np.sum(np.array(np.shape(x)) > 1)}D Parameter Space')
    ax.set_xlabel('X-axis')
    ax.set_ylabel('Y-axis')
    ax.set_zlabel('Z-axis')

    # Show plot
    plt.show()

# Generate data for 3D space
points = np.linspace(0, 1, 10)
x, y, z = np.meshgrid(points, 1, 1)
plot_points(x, y, z)
x, y, z = np.meshgrid(points, points, 1)
plot_points(x, y, z)
x, y, z = np.meshgrid(points, points, points)
plot_points(x, y, z)
```