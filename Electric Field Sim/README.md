# nanotech_capacitor (WORK FROM NANDANDIWAN)
 basic capacitor model, wip

## TODO:
### Dielectric Sim work
1. Work on adding any further terms to the dielectric expansion to account for other factors affecting polarization
2. Create a test 4-material stack and check if the swapping non-boundary doesn't affect K value

### Heat Disspation Work
1. Assume voltage applies is V. Research how much current flows through the chip traces when using the buck-boose converter. Trace dimensions: 50nm thickness, 100micron width, inch long

### Breakdown Voltage Sim
1. Create sim for breakdown voltage of caoacitor

### Capacitor Sim work 
#### Capacitor Sim visuals 
1. Fix poisson method so the slopes of potential matches with permitvitty (high permitivity = low slope on V-x graph) currently it is the opposite 
2. Visualization portion mostly done, need to add in potential and energy density --Navya
3. Better solver param config, separate sections 

#### Capacitor sim nanotubes 
Plan to implement CNTs in sim:
To implement CNTs in the capacitor sim we need to self consistently solve time independent Schrodinger equation and the non linear poisson's equation 
- [charge transport in graphene nanoribbons](https://pubs.aip.org/aip/jap/article/106/2/024509/393619/Modeling-charge-transport-in-graphene-nanoribbons)
- [CNT FET sim](https://link.springer.com/article/10.1007/s10825-006-8836-z)
 
1. Write poisson solver in 2D (done)
2. Test 1D hamiltonian as per datta book 
3. Self consistent procedure (1D/ 2D)
4. 2D Hamiltonian
5. Solve all self consistently 
6.  

#### Distant future
1. Implement interface physics 
2. Create a responsive dielectric - read Yu and Cardono dielectric function section?
3. IMPORTANT: improve the solver for high dielectrics, the current method (jacobi iteration) does not work well for high $k$ dielectrics
4. Incorporate more accurate boundary conditions


# References
## Jacobi Method for Poisson's Equation

Derivation of Jacobi method for Poisson's equation. Essentially we split our 3D space into a bunch of little lattice points. Each point has its own electric potential. To make computation easier we will define our Epsilon lattice points to be halfway in between the electric potential lattice points. Each little cube made up by lattice points is called a cell. 

The Jacobi method is an iterative method that converges to a solution. 
### Manipulating Poisson's equation 
We start off with the generalized Poisson's equation:

$$\nabla \cdot (\epsilon \nabla V)  = -\frac{\rho}{\epsilon_0}$$

We then volume integral over the cell and use the divergence theorem. 

$$ \int_V \nabla \cdot (\epsilon \nabla V) = \oint_S \epsilon\frac{\partial V}{\partial n} \,dA = -\frac{Q}{\epsilon_0}$$

We evaluate the surface integral over the cell keep in mind the signs on the integrals.

$$ \int_{S1}\epsilon \frac{\partial V}{\partial x}\,dy\,dz - 
\int_{S2}\epsilon\frac{\partial V}{\partial x}\,dy\,dz + 
\int_{S3}\epsilon\frac{\partial V}{\partial y}\,dx\,dz - 
\int_{S4}\epsilon\frac{\partial V}{\partial y}\,dx\,dz\,\, +
\int_{S5}\epsilon\frac{\partial V}{\partial z} \,dx\,dy - 
\int_{S6}\epsilon\frac{\partial V}{\partial z}\,dx\,dy = -\frac{Q(i,j,k)}{\epsilon_0} $$

Consider the first term:

$$ \int_{S1}\epsilon\frac{\partial V}{\partial x}\,dy\,dz = \int_{-\frac{b}{2}}^{\frac{b}{2}}\int_{-\frac{c}{2}}^{\frac{c}{2}}\epsilon\frac{\partial V}{\partial x} \,dy\,dz $$

To find the gradient of $V$, we use a positive difference while the permittivity is taken as the average of the four points closest to the face.

$$\int_{S1} = \frac{bc}{4}(\epsilon(i, j,k) + \epsilon(i, j - 1,k) + \epsilon(i, j,k-1) + \epsilon(i, j-1,k-1)) \cdot \left(\frac{V(i+1, j, k) - V(i)}{a}\right)$$

We can do something similar for the remaining 6 sides:

$$
\begin{aligned}
\int_{S2} &= \frac{bc}{4}(\epsilon(i-1, j,k) + \epsilon(i-1, j - 1,k) + \epsilon(i-1, j,k-1) + \epsilon(i-1, j-1,k-1)) \\
          &\quad \cdot \left(\frac{V(i-1, j, k) - V(i,j,k)}{a}\right)
\end{aligned}
$$


$$
\begin{aligned}
\int_{S3} &= \frac{ac}{4}(\epsilon(i, j,k) + \epsilon(i-1, j,k) + \epsilon(i, j,k-1) + \epsilon(i-1, j,k-1)) \\
          &\quad \cdot \left(\frac{V(i, j+1, k) - V(i,j,k)}{b}\right)
\end{aligned}
$$


$$
\begin{aligned}
\int_{S4} &= \frac{ac}{4}(\epsilon(i, j-1,k) + \epsilon(i-1, j-1,k) + \epsilon(i, j-1,k-1) + \epsilon(i-1, j-1,k-1)) \\
          &\quad \cdot \left(\frac{V(i, j-1, k) - V(i,j,k)}{b}\right)
\end{aligned}
$$


$$
\begin{aligned}
\int_{S5} &= \frac{ab}{4}(\epsilon(i, j,k) + \epsilon(i-1, j,k) + \epsilon(i, j-1,k) + \epsilon(i-1, j-1,k)) \\
          &\quad \cdot \left(\frac{V(i, j, k+1) - V(i,j,k)}{c}\right)
\end{aligned}
$$


$$
\begin{aligned}
\int_{S6} &= \frac{ab}{4}(\epsilon(i, j,k-1) + \epsilon(i-1, j,k-1) + \epsilon(i, j-1,k-1) + \epsilon(i-1, j-1,k-1)) \\
          &\quad \cdot \left(\frac{V(i, j, k-1) - V(i,j,k)}{c}\right) 
\end{aligned}
$$


### Combined Equation

Adding all terms and setting equal them equal to $-\frac{Q(i,j,k)}{\epsilon_0}$:

$$
\begin{aligned}
-\frac{Q(i,j,k)}{\epsilon_0} &= a_0 V(i,j,k) + a_1 V(i+1,j,k) + a_2 V(i-1,j,k) + a_3 V(i,j+1,k) \\
                             &\quad + a_4 V(i,j-1,k) + a_5 V(i,j,k+1) + a_6 V(i,j,k-1) 
\end{aligned}
$$


Solving for $V(i,j,k)$:

$$
\begin{aligned}
V(i,j,k) &= \frac{1}{a_0}\left(\frac{Q(i,j,k)}{\epsilon_0} + a_1 V(i+1,j,k) + a_2 V(i-1,j,k) + a_3 V(i,j+1,k) \right. \\
         &\quad \left. + a_4 V(i,j-1,k) + a_5 V(i,j,k+1) + a_6 V(i,j,k-1)\right)
\end{aligned}
$$


### Residual Calculation
Now we have to find the change in $V(i,j,k)$ every iteration. This quantity is the residual. 

$$
\begin{aligned}
R &= \frac{1}{a_0}\left(\frac{Q(i,j,k)}{\epsilon_0} + a_1 V(i+1,j,k) + a_2 V(i-1,j,k) + a_3 V(i,j+1,k) \right. \\
  &\quad \left. + a_4 V(i,j-1,k) + a_5 V(i,j,k+1) + a_6 V(i,j,k-1)\right) - V(i,j,k)
\end{aligned}
$$


$$V^{n+1} = V^n +  R$$

### Coefficients for Iterations
For reference:

$$
\begin{aligned}
a_0 &= -\frac{bc}{4a} (\epsilon(i, j, k) + \epsilon(i, j-1, k) + \epsilon(i, j, k-1) + \epsilon(i, j-1, k-1)) \\
    &\quad -\frac{bc}{4a} (\epsilon(i-1, j, k) + \epsilon(i-1, j-1, k) + \epsilon(i-1, j, k-1) + \epsilon(i-1, j-1, k-1)) \\
    &\quad -\frac{ac}{4b} (\epsilon(i, j, k) + \epsilon(i-1, j, k) + \epsilon(i, j, k-1) + \epsilon(i-1, j, k-1)) \\
    &\quad -\frac{ac}{4b} (\epsilon(i, j-1, k) + \epsilon(i-1, j-1, k) + \epsilon(i, j-1, k-1) + \epsilon(i-1, j-1, k-1)) \\
    &\quad -\frac{ab}{4c} (\epsilon(i, j, k) + \epsilon(i-1, j, k) + \epsilon(i, j-1, k) + \epsilon(i-1, j-1, k)) \\
    &\quad -\frac{ab}{4c} (\epsilon(i, j, k-1) + \epsilon(i-1, j, k-1) + \epsilon(i, j-1, k-1) + \epsilon(i-1, j-1, k-1))
\end{aligned}
$$


$$
\begin{aligned}
a_1 &= \frac{bc}{4a} (\epsilon(i, j, k) + \epsilon(i, j-1, k) + \epsilon(i, j, k-1) + \epsilon(i, j-1, k-1))
\end{aligned}
$$


$$
\begin{aligned}
a_2 &= \frac{bc}{4a} (\epsilon(i-1,j,k) + \epsilon(i-1,j-1,k) + \epsilon(i-1,j,k-1) + \epsilon(i-1,j-1,k-1))
\end{aligned}
$$


$$
\begin{aligned}
a_3 &= \frac{ac}{4b} (\epsilon(i,j,k) + \epsilon(i-1,j,k) + \epsilon(i,j,k-1) + \epsilon(i-1,j,k-1))
\end{aligned}
$$


$$
\begin{aligned}
a_4 &= -\frac{ac}{4b} (\epsilon(i,j-1,k) + \epsilon(i-1,j-1,k) + \epsilon(i,j-1,k-1) + \epsilon(i-1,j-1,k-1))
\end{aligned}
$$


$$
\begin{aligned}
a_5 &= \frac{ab}{4c} (\epsilon(i,j,k) + \epsilon(i-1,j,k) + \epsilon(i,j-1,k) + \epsilon(i-1,j-1,k))
\end{aligned}
$$


$$
\begin{aligned}
a_6 &= -\frac{ab}{4c} (\epsilon(i,j,k-1) + \epsilon(i-1,j,k-1) + \epsilon(i,j-1,k-1) + \epsilon(i-1,j-1,k-1))
\end{aligned}
$$
