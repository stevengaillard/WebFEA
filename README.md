# WebFEA - 2D Frame Analysis

**[English](#) | [繁體中文](./README.zh-TW.md)**

---

## Overview

**WebFEA** is a comprehensive web-based structural engineering application for the analysis and design of 2D frame structures and truss systems. The tool provides real-time finite element analysis, interactive canvas visualization, and automated structural calculations with support for complex loading conditions including distributed loads, point loads, settlements, and element-specific loads with local/global coordinate systems.

---

## Core Features

### Calculation Engine

* **Direct Stiffness Method**: Classical matrix structural analysis with 3 DOF per node (Δx, Δy, θz)
* **Element Types**: 
  - Frame elements with bending, axial, and shear stiffness
  - Truss elements (pin-connected, axial force only)
* **Load Types**:
  - Nodal point loads (Fx, Fy, M)
  - Support settlements (prescribed displacements)
  - Element point loads (at any location along span)
  - Distributed loads (uniform or trapezoidal, partial or full span)
  - Load angle reference: Global or Local coordinate system
* **Support Conditions**: Fixed, pinned, and roller supports with rotatable angle
* **Results**:
  - Node displacements (Δx, Δy, θz)
  - Support reactions (Rx, Ry, Rm)
  - Member internal forces (Axial N, Shear V, Moment M)
  - Deformed shape visualization
  - Internal force diagrams (BMD, SFD, AFD)

### User Interface

* **Dual Language Support**: Instant toggling between English and Traditional Chinese (繁體中文)
* **Real-time Visualization**:
  - Interactive canvas with pan and zoom controls
  - Grid system with dynamic subdivision (binary/decimal steps)
  - Live force diagrams overlaid on structure
  - Deformed shape with exaggerated scale
  - Hover tooltips showing internal forces at any point
* **Interactive Tools**:
  - Node placement with grid snapping
  - Element creation (click start-end nodes)
  - Support assignment with angle control
  - Load application with visual feedback
  - Selection mode for editing/deletion
* **Section Library**:
  - Pre-defined standard sections (Concrete, Steel I-beams, Pipes)
  - Custom section calculator with 7 shape types
  - Automatic property calculation (Area, Moment of Inertia)
  - Material database (Elastic modulus E)
* **Advanced Features**:
  - Undo/Redo functionality (Ctrl+Z, Ctrl+Y)
  - Auto-save to browser local storage
  - Project file export/import (JSON format)
  - Demo models (Portal frame, Truss bridge)
  - Dark/Light theme toggle
  - Display options panel (diagrams, reactions, scales)

---

## Structural Analysis Theory

### 1. Coordinate Systems

#### Global Coordinate System
- **X-axis**: Horizontal, positive to the right
- **Y-axis**: Vertical, positive upward
- **Origin**: User-defined (typically at bottom-left of canvas)

#### Local Element Coordinate System
- **x-axis**: Along element from node i to node j
- **y-axis**: Perpendicular to element (right-hand rule)
- **Transformation angle** α = atan2(Δy, Δx)

---

### 2. Element Stiffness Matrix

#### 2.1 Frame Element (6×6 Local Stiffness)

For a frame element with:
- Length: L (m)
- Elastic modulus: E (kPa)
- Cross-sectional area: A (m²)
- Moment of inertia: I (m⁴)

**Local DOF ordering**: [u₁, v₁, θ₁, u₂, v₂, θ₂]

**Local stiffness matrix**:

$$
\mathbf{k}_{\text{local}} = \begin{bmatrix}
k_1 & 0 & 0 & -k_1 & 0 & 0 \\
0 & k_2 & k_3 & 0 & -k_2 & k_3 \\
0 & k_3 & k_4 & 0 & -k_3 & k_5 \\
-k_1 & 0 & 0 & k_1 & 0 & 0 \\
0 & -k_2 & -k_3 & 0 & k_2 & -k_3 \\
0 & k_3 & k_5 & 0 & -k_3 & k_4
\end{bmatrix}
$$

where:

$$
k_1 = \frac{EA}{L}
$$

$$
k_2 = \frac{12EI}{L^3}
$$

$$
k_3 = \frac{6EI}{L^2}
$$

$$
k_4 = \frac{4EI}{L}
$$

$$
k_5 = \frac{2EI}{L}
$$

**Physical meaning**:
- k₁: Axial stiffness
- k₂: Transverse shear stiffness
- k₃: Shear-moment coupling
- k₄: Rotational stiffness at near end
- k₅: Rotational stiffness at far end (half of k₄ due to symmetry)

#### 2.2 Truss Element (6×6 Local Stiffness)

For truss elements, bending stiffness is zero:

$$
k_2 = k_3 = k_4 = k_5 = 0
$$

**Singularity patch**: To prevent zero diagonal in rotational DOF:

$$
k_{\text{local}}[2,2] = k_{\text{local}}[5,5] = 1.0 \text{ (if } |k_4| < 10^{-9}\text{)}
$$

#### 2.3 Transformation Matrix (6×6)

**Direction cosines**:

$$
c = \frac{\Delta x}{L} = \cos \alpha
$$

$$
s = \frac{\Delta y}{L} = \sin \alpha
$$

**Transformation matrix**:

$$
\mathbf{T} = \begin{bmatrix}
c & s & 0 & 0 & 0 & 0 \\
-s & c & 0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & c & s & 0 \\
0 & 0 & 0 & -s & c & 0 \\
0 & 0 & 0 & 0 & 0 & 1
\end{bmatrix}
$$

#### 2.4 Global Stiffness Matrix

$$
\mathbf{k}_{\text{global}} = \mathbf{T}^T \mathbf{k}_{\text{local}} \mathbf{T}
$$

**Assembly**: Each element's global stiffness is assembled into the structure stiffness matrix **K** at DOF locations:

- Node i: [3i, 3i+1, 3i+2]
- Node j: [3j, 3j+1, 3j+2]

---

### 3. Load Vector Assembly

#### 3.1 Nodal Point Loads

Applied forces are added directly to the global force vector **F**:

$$
F[3n] \mathrel{+}= F_x
$$

$$
F[3n+1] \mathrel{+}= F_y
$$

$$
F[3n+2] \mathrel{+}= M_z
$$

where n is the node index.

#### 3.2 Element Loads (Fixed-End Actions)

Element loads (distributed or point on span) are converted to equivalent nodal forces using **fixed-end action** formulas, then applied to the global force vector.

##### 3.2.1 Point Load on Span

**Input parameters**:
- Magnitude: P (kN)
- Angle: θ (degrees, measured from horizontal)
- Distance from node i: a (m)
- Element length: L (m)
- Angle reference: "global" or "local"

**Step 1: Determine global angle**

If angleRef = "local":
$$
\theta_{\text{global}} = \theta_{\text{beam}} + \theta_{\text{input}}
$$

where:
$$
\theta_{\text{beam}} = \arctan\left(\frac{\Delta y}{\Delta x}\right)
$$

If angleRef = "global":
$$
\theta_{\text{global}} = \theta_{\text{input}}
$$

**Step 2: Decompose to global components**

$$
F_x^{\text{global}} = P \cos(\theta_{\text{global}})
$$

$$
F_y^{\text{global}} = P \sin(\theta_{\text{global}})
$$

**Step 3: Transform to local beam coordinates**

$$
P_x = F_x^{\text{global}} \cos(\theta_{\text{beam}}) + F_y^{\text{global}} \sin(\theta_{\text{beam}})
$$

$$
P_y = -F_x^{\text{global}} \sin(\theta_{\text{beam}}) + F_y^{\text{global}} \cos(\theta_{\text{beam}})
$$

**Step 4: Calculate distance from far end**

$$
b = L - a
$$

**Step 5: Fixed-end actions (transverse component P_y)**

**Moments** (sign convention: CCW positive):

$$
M_1 = \frac{P_y \cdot a \cdot b^2}{L^2}
$$

$$
M_2 = -\frac{P_y \cdot a^2 \cdot b}{L^2}
$$

**Shear forces**:

$$
V_1 = \frac{P_y \cdot b^2 (3a + b)}{L^3}
$$

$$
V_2 = \frac{P_y \cdot a^2 (a + 3b)}{L^3}
$$

**Axial forces** (linear distribution):

$$
N_1 = -P_x \cdot \frac{b}{L}
$$

$$
N_2 = -P_x \cdot \frac{a}{L}
$$

**Step 6: Assemble local fixed-end force vector**

$$
\mathbf{f}_{\text{FE,local}} = \begin{bmatrix} N_1 \\ V_1 \\ M_1 \\ N_2 \\ V_2 \\ M_2 \end{bmatrix}
$$

**Step 7: Transform to global**

$$
\mathbf{f}_{\text{FE,global}} = \mathbf{T}^T \mathbf{f}_{\text{FE,local}}
$$

**Step 8: Apply to global force vector**

$$
F[3i] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[0]
$$

$$
F[3i+1] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[1]
$$

$$
F[3i+2] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[2]
$$

$$
F[3j] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[3]
$$

$$
F[3j+1] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[4]
$$

$$
F[3j+2] \mathrel{+}= \mathbf{f}_{\text{FE,global}}[5]
$$

##### 3.2.2 Distributed Load (Trapezoidal)

**Input parameters**:
- Start magnitude: w₁ (kN/m)
- End magnitude: w₂ (kN/m)
- Start distance: d₁ (m)
- End distance: d₂ (m)
- Angle: θ (degrees)
- Angle reference: "global" or "local"

**Integration method**: Numerical integration using **Simpson's rule** (10 steps)

**Step 1: Determine global angle** (same as point load)

**Step 2: Calculate step size**

$$
\Delta L = d_2 - d_1
$$

$$
h = \frac{\Delta L}{n} \quad (n = 10 \text{ steps})
$$

**Step 3: Loop through integration points**

For i = 0 to n:

$$
x = d_1 + i \cdot h
$$

**Interpolation parameter**:

$$
s = \frac{x - d_1}{\Delta L}
$$

**Load intensity at x**:

$$
w_y(x) = w_1 (1 - s) + w_2 \cdot s
$$

$$
w_x(x) = w_1 (1 - s) + w_2 \cdot s
$$

after decomposing angle:

$$
n_x = \cos(\theta_{\text{global}}), \quad n_y = \sin(\theta_{\text{global}})
$$

$$
w_x(x) = w(x) \cdot (n_x \cos\theta_{\text{beam}} + n_y \sin\theta_{\text{beam}})
$$

$$
w_y(x) = w(x) \cdot (-n_x \sin\theta_{\text{beam}} + n_y \cos\theta_{\text{beam}})
$$

**Simpson's weight**:

$$
\text{weight} = \begin{cases}
1 & \text{if } i = 0 \text{ or } i = n \\
4 & \text{if } i \text{ is odd} \\
2 & \text{if } i \text{ is even}
\end{cases}
$$

$$
w_{\text{eff}} = \frac{\text{weight} \cdot h}{3}
$$

**Differential fixed-end actions** (treating dw as point load):

Distance from far end:

$$
b = L - x
$$

**Differential moments**:

$$
dM_1 = \frac{w_y(x) \cdot x \cdot b^2}{L^2} \cdot w_{\text{eff}}
$$

$$
dM_2 = -\frac{w_y(x) \cdot x^2 \cdot b}{L^2} \cdot w_{\text{eff}}
$$

**Differential shear**:

$$
dV_1 = \frac{w_y(x) \cdot b^2 (3x + b)}{L^3} \cdot w_{\text{eff}}
$$

$$
dV_2 = \frac{w_y(x) \cdot x^2 (x + 3b)}{L^3} \cdot w_{\text{eff}}
$$

**Differential axial**:

$$
dN_1 = -w_x(x) \cdot \frac{b}{L} \cdot w_{\text{eff}}
$$

$$
dN_2 = -w_x(x) \cdot \frac{x}{L} \cdot w_{\text{eff}}
$$

**Accumulate**:

$$
N_1 \mathrel{+}= dN_1, \quad V_1 \mathrel{+}= dV_1, \quad M_1 \mathrel{+}= dM_1
$$

$$
N_2 \mathrel{+}= dN_2, \quad V_2 \mathrel{+}= dV_2, \quad M_2 \mathrel{+}= dM_2
$$

**Transform and apply** (same as Steps 6-8 in point load)

#### 3.3 Support Settlements

For prescribed displacements (Δx, Δy, θ) at support nodes:

**Add to known displacement vector**:

$$
D_{\text{known}}[3n] = \Delta x
$$

$$
D_{\text{known}}[3n+1] = \Delta y
$$

$$
D_{\text{known}}[3n+2] = \theta
$$

**Mark DOF as restrained**:

$$
\text{restrainedDofSet.add}(3n), \quad \text{restrainedDofSet.add}(3n+1), \quad \text{restrainedDofSet.add}(3n+2)
$$

---

### 4. System of Equations

#### 4.1 Partitioning

**Free DOFs**: DOFs without support restraints

**Restrained DOFs**: DOFs with supports or prescribed displacements

**Partition global stiffness**:

$$
\begin{bmatrix}
\mathbf{K}_{ff} & \mathbf{K}_{fr} \\
\mathbf{K}_{rf} & \mathbf{K}_{rr}
\end{bmatrix}
\begin{bmatrix}
\mathbf{D}_f \\
\mathbf{D}_r
\end{bmatrix}
=
\begin{bmatrix}
\mathbf{F}_f \\
\mathbf{F}_r
\end{bmatrix}
$$

#### 4.2 Effective Force Vector

Account for prescribed displacements:

$$
\mathbf{F}_{f,\text{eff}} = \mathbf{F}_f - \mathbf{K}_{fr} \mathbf{D}_r
$$

#### 4.3 Solution for Free DOFs

$$
\mathbf{K}_{ff} \mathbf{D}_f = \mathbf{F}_{f,\text{eff}}
$$

**Solver**: Gaussian elimination with partial pivoting

**Forward elimination**:

For i = 0 to n-1:

1. Find pivot row (maximum |K[k][i]| for k ≥ i)
2. Swap rows i and maxRow
3. Check for singularity: |K[i][i]| < 10⁻¹⁰ → Error
4. Eliminate column below pivot:

$$
\text{factor} = \frac{K[k][i]}{K[i][i]}
$$

$$
K[k][j] \mathrel{-}= \text{factor} \cdot K[i][j] \quad \forall j \geq i
$$

**Back substitution**:

For i = n-1 to 0:

$$
D_f[i] = \frac{F_{f,\text{eff}}[i] - \sum_{j=i+1}^{n-1} K[i][j] D_f[j]}{K[i][i]}
$$

#### 4.4 Complete Displacement Vector

$$
\mathbf{D} = \begin{cases}
D_f[k] & \text{if DOF is free} \\
D_r[k] & \text{if DOF is restrained}
\end{cases}
$$

---

### 5. Member Forces Recovery

For each element e:

**Step 1: Extract global displacements**

$$
\mathbf{d}_{\text{global}} = \begin{bmatrix}
D[3i] \\ D[3i+1] \\ D[3i+2] \\ D[3j] \\ D[3j+1] \\ D[3j+2]
\end{bmatrix}
$$

**Step 2: Transform to local coordinates**

$$
\mathbf{u}_{\text{local}} = \mathbf{T} \cdot \mathbf{d}_{\text{global}}
$$

**Step 3: Calculate element forces in local system**

$$
\mathbf{f}_{\text{local}} = \mathbf{k}_{\text{local}} \cdot \mathbf{u}_{\text{local}}
$$

**Step 4: Subtract fixed-end forces** (if element loads present)

$$
\mathbf{f}_{\text{local}} \mathrel{-}= \mathbf{f}_{\text{FE,local}}
$$

**Physical interpretation**:

$$
\mathbf{f}_{\text{local}} = \begin{bmatrix}
N_1 \\ V_1 \\ M_1 \\ N_2 \\ V_2 \\ M_2
\end{bmatrix}
$$

where:
- N₁, N₂: Axial forces (positive = tension)
- V₁, V₂: Shear forces
- M₁, M₂: Bending moments (CCW positive)

**Sign convention for output**:
- **Axial**: N₂ (force at node j, tension positive)
- **Shear at start**: V₁
- **Shear at end**: V₂
- **Moment at start**: M₁
- **Moment at end**: M₂

**Step 5: Transform back to global for reactions**

$$
\mathbf{f}_{\text{global}} = \mathbf{T}^T \cdot \mathbf{f}_{\text{local}}
$$

$$
R_{\text{accum}}[3i] \mathrel{+}= \mathbf{f}_{\text{global}}[0]
$$

$$
R_{\text{accum}}[3i+1] \mathrel{+}= \mathbf{f}_{\text{global}}[1]
$$

$$
R_{\text{accum}}[3i+2] \mathrel{+}= \mathbf{f}_{\text{global}}[2]
$$

$$
R_{\text{accum}}[3j] \mathrel{+}= \mathbf{f}_{\text{global}}[3]
$$

$$
R_{\text{accum}}[3j+1] \mathrel{+}= \mathbf{f}_{\text{global}}[4]
$$

$$
R_{\text{accum}}[3j+2] \mathrel{+}= \mathbf{f}_{\text{global}}[5]
$$

---

### 6. Support Reactions

**Final reactions** (equilibrium):

$$
\mathbf{R} = \mathbf{R}_{\text{accum}} - \mathbf{F}_{\text{applied}}
$$

For each support node n:

$$
R_x = R_{\text{accum}}[3n] - F_{\text{applied}}[3n]
$$

$$
R_y = R_{\text{accum}}[3n+1] - F_{\text{applied}}[3n+1]
$$

$$
R_m = R_{\text{accum}}[3n+2] - F_{\text{applied}}[3n+2]
$$

where **F_applied** includes only direct nodal loads, not element-derived fixed-end forces.

---

### 7. Internal Forces at Any Point

To calculate M(x), V(x), N(x) at distance x from node i:

**Initialization** (from nodal forces):

$$
M(x) = -M_1 + V_1 \cdot x
$$

$$
V(x) = V_1
$$

$$
N(x) = N_1
$$

**Add contribution from loads between 0 and x**:

For each element load:

##### Point Load at distance a

If x > a:

$$
M(x) \mathrel{+}= P_y \cdot (x - a)
$$

$$
V(x) \mathrel{+}= P_y
$$

$$
N(x) \mathrel{+}= P_x
$$

##### Distributed Load from d₁ to d₂

Active segment: [max(d₁, 0), min(d₂, x)]

If end > start:

$$
\text{len} = \text{end} - \text{start}
$$

**Interpolated intensities**:

$$
w_{\text{start}} = w_1 + (w_2 - w_1) \frac{\text{start} - d_1}{d_2 - d_1}
$$

$$
w_{\text{end}} = w_1 + (w_2 - w_1) \frac{\text{end} - d_1}{d_2 - d_1}
$$

**Resultant force**:

$$
W = \frac{w_{\text{start}} + w_{\text{end}}}{2} \cdot \text{len}
$$

**Centroid distance from "start"**:

$$
\bar{x}_{\text{local}} = \frac{\text{len} \cdot (w_{\text{start}} + 2w_{\text{end}})}{3(w_{\text{start}} + w_{\text{end}})}
$$

**Lever arm to x**:

$$
\text{lever} = (x - \text{start}) - \bar{x}_{\text{local}}
$$

**Decompose to components**:

$$
W_x = W \cos(\theta_{\text{rel}})
$$

$$
W_y = W \sin(\theta_{\text{rel}})
$$

where:

$$
\theta_{\text{rel}} = \theta_{\text{global}} - \theta_{\text{beam}}
$$

**Update**:

$$
M(x) \mathrel{+}= W_y \cdot \text{lever}
$$

$$
V(x) \mathrel{+}= W_y
$$

$$
N(x) \mathrel{+}= W_x
$$

---

### 8. Deflection (Transverse Displacement)

**Cubic Hermite interpolation** (from nodal displacements):

$$
v(x) = v_1 H_1(t) + \theta_1 L \cdot H_2(t) + v_2 H_3(t) + \theta_2 L \cdot H_4(t)
$$

where:

$$
t = \frac{x}{L}
$$

**Hermite shape functions**:

$$
H_1(t) = 1 - 3t^2 + 2t^3
$$

$$
H_2(t) = t(1 - 2t + t^2) = t - 2t^2 + t^3
$$

$$
H_3(t) = 3t^2 - 2t^3
$$

$$
H_4(t) = t^2(t - 1) = t^3 - t^2
$$

**Relative deflection** (subtract rigid body motion):

$$
v_{\text{linear}}(t) = v_1(1 - t) + v_2 \cdot t
$$

$$
v_{\text{disp}}(x) = v(x) - v_{\text{linear}}(x)
$$

This isolates the bending deformation from rigid translation.

---

## Section Property Calculation

### Standard Shapes

#### 1. Rectangle

**Inputs**: Depth d (mm), Width b (mm)

**Formulas**:

$$
A = b \cdot d \quad (\text{m}^2)
$$

$$
I = \frac{b \cdot d^3}{12} \quad (\text{m}^4)
$$

#### 2. Hollow Rectangle (Tube)

**Inputs**: Outer d, b (mm), Wall thickness t (mm)

**Inner dimensions**:

$$
d_i = d - 2t, \quad b_i = b - 2t
$$

**Formulas**:

$$
A = bd - b_i d_i
$$

$$
I = \frac{bd^3 - b_i d_i^3}{12}
$$

#### 3. Circle

**Input**: Diameter d (mm)

**Formulas**:

$$
A = \frac{\pi d^2}{4}
$$

$$
I = \frac{\pi d^4}{64}
$$

#### 4. Hollow Circle (Pipe)

**Inputs**: Outer diameter d (mm), Wall thickness t (mm)

**Inner diameter**:

$$
d_i = d - 2t
$$

**Formulas**:

$$
A = \frac{\pi (d^2 - d_i^2)}{4}
$$

$$
I = \frac{\pi (d^4 - d_i^4)}{64}
$$

#### 5. I-Shape (Wide Flange)

**Inputs**: Depth d, Flange width b, Web thickness t_w, Flange thickness t_f (mm)

**Inner dimensions**:

$$
h_{\text{inner}} = d - 2t_f
$$

$$
b_{\text{inner}} = b - t_w
$$

**Formulas**:

$$
A = 2b t_f + h_{\text{inner}} t_w
$$

$$
I = \frac{b d^3 - b_{\text{inner}} h_{\text{inner}}^3}{12}
$$

#### 6. C-Shape (Channel)

**Same formulas as I-Shape** (symmetric approximation)

#### 7. Triangle

**Inputs**: Base b, Height d (mm)

**Formulas**:

$$
A = \frac{b \cdot d}{2}
$$

$$
I = \frac{b \cdot d^3}{36}
$$

---

## Visualization & Rendering

### Canvas Coordinate System

**Screen to World**:

$$
x_{\text{world}} = \frac{x_{\text{screen}} - \text{pan}_x}{\text{zoom}}
$$

$$
y_{\text{world}} = \frac{\text{pan}_y - y_{\text{screen}}}{\text{zoom}}
$$

**World to Screen**:

$$
x_{\text{screen}} = \text{pan}_x + x_{\text{world}} \cdot \text{zoom}
$$

$$
y_{\text{screen}} = \text{pan}_y - y_{\text{world}} \cdot \text{zoom}
$$

### Grid System

**Dynamic step calculation**:

High zoom (zoom ≥ 40 px/m):

$$
\text{step} = \frac{0.5}{2^{\lfloor \log_2(\text{zoom}/40) \rfloor}}
$$

Low zoom (zoom < 40 px/m):

$$
\text{rawStep} = \frac{50}{\text{zoom}}
$$

$$
\text{magnitude} = 10^{\lfloor \log_{10}(\text{rawStep}) \rfloor}
$$

$$
\text{residual} = \frac{\text{rawStep}}{\text{magnitude}}
$$

$$
\text{step} = \begin{cases}
10 \cdot \text{magnitude} & \text{if residual} > 5 \\
5 \cdot \text{magnitude} & \text{if residual} > 2 \\
2 \cdot \text{magnitude} & \text{if residual} > 1 \\
\text{magnitude} & \text{otherwise}
\end{cases}
$$

### Diagram Scaling

**Global scales** (computed from all elements):

$$
\text{scale}_M = \frac{1.5}{\max |M(x)|}
$$

$$
\text{scale}_V = \frac{1.5}{\max |V(x)|}
$$

$$
\text{scale}_N = \frac{1.5}{\max |N(x)|}
$$

$$
\text{scale}_\delta = \frac{1.0}{\max |v(x)|}
$$

**Display scale** (user-adjustable):

$$
\text{offset}_{\text{pixels}} = \text{value} \times \text{scale} \times \text{diaScale} \times 40
$$

### Deformed Shape Rendering

**Curved beam** (40 steps along length):

For each step i = 0 to 40:

$$
t = \frac{i}{40}
$$

$$
x = t \cdot L
$$

**Deflection**:

$$
v_{\text{disp}}(x) = \text{(Hermite interpolation)}
$$

**Screen offset** (perpendicular to beam):

$$
\text{offset}_{\text{screen}} = -v_{\text{disp}}(x) \cdot \text{defScale} \cdot \text{zoom}
$$

**Normal vector** (perpendicular direction):

For horizontal beams (|dy| ≤ |dx|):

$$
n_x = -\frac{dy}{L_{\text{screen}}}, \quad n_y = \frac{dx}{L_{\text{screen}}}
$$

For vertical beams (|dy| > |dx|):

$$
n_x = -\frac{dy}{L_{\text{screen}}}, \quad n_y = \frac{dx}{L_{\text{screen}}}
$$

**Curved point position**:

$$
x_{\text{screen}} = x_{1,\text{screen}} + t \cdot \Delta x_{\text{screen}} + n_x \cdot \text{offset}_{\text{screen}}
$$

$$
y_{\text{screen}} = y_{1,\text{screen}} + t \cdot \Delta y_{\text{screen}} + n_y \cdot \text{offset}_{\text{screen}}
$$

---

## Units Convention

| Parameter | Unit | Description |
|-----------|------|-------------|
| **Length** | m | Node coordinates, span lengths |
| **Small Length** | cm | Section dimensions, thickness |
| **Force** | kN | Loads, reactions, internal forces |
| **Moment** | kN·m | Bending moments |
| **Distributed Load** | kN/m | Line loads |
| **Stress** | kPa | Elastic modulus E |
| **Area** | m² | Cross-sectional area |
| **Inertia** | m⁴ | Moment of inertia |
| **Angle** | degrees | Load angles, support rotation |

---

## Browser Compatibility

* **Chrome/Edge**: v90+
* **Firefox**: v88+
* **Safari**: v14+
* **Mobile**: iOS Safari, Chrome Mobile (touch controls enabled)

**Required Features**:
- HTML5 Canvas
- ES6 JavaScript (arrow functions, destructuring, modules)
- CSS Grid and Flexbox
- LocalStorage API
- FileReader API (for project loading)

---

## Known Limitations

1. **2D Analysis Only**: Out-of-plane effects not considered
2. **Linear Elastic**: No material nonlinearity or plasticity
3. **Small Deformations**: Geometric nonlinearity (P-Δ effects) not included
4. **No Automatic Meshing**: User must discretize curved members manually
5. **No Time History**: Static analysis only
6. **Limited Load Types**: No temperature loads, no moving loads
7. **No Design Code Checks**: Pure analysis tool (no ACI/Eurocode compliance checks)

---

## References

### Textbooks
* **Matrix Structural Analysis** by William McGuire, Richard H. Gallagher, Ronald D. Ziemian (2nd Edition)
* **Structural Analysis** by Aslam Kassimali (6th Edition)
* **Finite Element Procedures** by Klaus-Jürgen Bathe

### Standards
* **AISC Steel Construction Manual** (15th Edition) - Section properties
* **ACI 318-19** - Concrete design (referenced for material properties only)

### Algorithms
* **Gaussian Elimination with Partial Pivoting** - Numerical Recipes (Press et al.)
* **Hermite Interpolation** - Standard finite element formulation

---

## Support

**Developer**: gacchuguts@gmail.com

---

**Disclaimer**: This software is provided as an educational and preliminary design tool. Users must verify all results independently and ensure compliance with applicable codes and standards. The author assumes no liability for errors, omissions, or consequences of use. Always consult a licensed professional engineer for final design approval.