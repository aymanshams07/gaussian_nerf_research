# gaussian_nerf_research
research and dev work

## three parts (NeRF, Gaussian Splatting, and Plane derivation):

---

# 1) Neural Radiance Fields (NeRF) — Derivation from Scratch
core concept : NeRF encodes a 3D scene as a continuous volumetric function that maps 3D coordinates and viewing directions to emitted color and density. A multilayer perceptron (MLP) learns this mapping from input photos. Rendering is achieved by integrating color and density along camera rays through the volume, simulating light transport.

Training and rendering process

During training, NeRF compares rendered pixels from random camera rays to their corresponding real images, optimizing network parameters through gradient descent. Once trained, the model can synthesize unseen perspectives by sampling new rays through the learned field. This enables smooth, high-fidelity camera motion around complex objects or scenes.

## Step 1: Scene Representation

We define a function:

```
F(x, d) → (c, σ)
```

* `x ∈ R^3`: 3D position
* `d ∈ R^3`: viewing direction
* `c ∈ R^3`: RGB color
* `σ ∈ R`: volume density

---

## Step 2: Ray Parameterization

A camera ray is defined as:

```
r(t) = o + t d
```

* `o`: camera origin
* `d`: direction
* `t`: depth along ray

---

## Step 3: Volume Rendering Equation

The rendered color of a pixel is:

```
C(r) = ∫_{t_n}^{t_f} T(t) σ(r(t)) c(r(t), d) dt
```

Where `T(t)` is the transmittance.

---

## Step 4: Transmittance

```
T(t) = exp(-∫_{t_n}^{t} σ(r(s)) ds)
```

Interpretation:

* Probability that light reaches point `t` without being absorbed.

---

## Step 5: Discretization

Sample points along the ray:

```
t1, t2, ..., tN
```

Define:

```
α_i = 1 - exp(-σ_i δ_i)
```

```
T_i = ∏_{j < i} (1 - α_j)
```

Final color:

```
C = Σ_i T_i α_i c_i
```

---

## Step 6: Neural Parameterization

We model:

```
F_θ(x, d) = (σ, c)
```

* Implemented using an MLP
* Uses positional encoding

---

## Key Insight

* NeRF models a **continuous volumetric field**
* Rendering is done via **integration along rays**

---

# 2) Gaussian Splatting — Derivation from Scratch

## Step 1: Scene Representation

Represent the scene as a sum of Gaussians:

```
G_i(x) = w_i * N(x | μ_i, Σ_i)
```

Where:

* `μ_i`: mean (center)
* `Σ_i`: covariance matrix
* `w_i`: weight / opacity

---

## Step 2: Gaussian Definition

```
N(x) = (1 / ((2π)^{3/2} |Σ|^{1/2})) * exp(-1/2 (x - μ)^T Σ^{-1} (x - μ))
```

---

## Step 3: Projection to Screen

We project 3D points to 2D:

```
x_screen = Π(x)
```

Covariance transforms as:

```
Σ_2D = J Σ_3D J^T
```

* `J`: Jacobian of projection

---

## Step 4: Rendering via Alpha Compositing

Sort Gaussians front-to-back:

```
C = Σ_i T_i α_i c_i
```

Where:

```
T_i = ∏_{j < i} (1 - α_j)
```

---

## Step 5: Key Insight

* Gaussian splatting = **explicit discrete approximation of radiance fields**
* No ray marching required
* Enables real-time rendering

---

# 3) Plane-Based Derivation (Interview-Style Question)

## Problem

Given a 3D Gaussian, derive its projection onto a 2D image plane.

---

## Step 1: Plane Equation

A plane is defined as:

```
n^T x + d = 0
```

* `n`: normal vector
* `d`: offset

---

## Step 2: Camera Projection

Projection function:

```
x_2D = Π(x)
```

We linearize around the Gaussian mean `μ`:

```
Π(x) ≈ Π(μ) + J (x - μ)
```

* `J`: Jacobian of projection

---

## Step 3: Gaussian Transformation

Original distribution:

```
x ~ N(μ, Σ)
```

Apply linear transform:

```
y = J x
```

Then:

```
y ~ N(Jμ, J Σ J^T)
```

---

## Step 4: Final 2D Gaussian

After projection:

```
μ_2D = Π(μ)
Σ_2D = J Σ J^T
```

So:

```
N(μ_2D, Σ_2D)
```

---

## Step 5: Geometric Interpretation

* 3D ellipsoid → 2D ellipse
* Orientation preserved via covariance
* Scale affected by depth and perspective

---

## Step 6: Why This Matters

This is the **core mathematical step in Gaussian splatting rendering**:

* Converts 3D Gaussians → 2D splats
* Enables rasterization instead of ray marching
* Critical for real-time performance on mobile devices

---

# Final Summary (Interview-Level Insight)

* **NeRF**:

  * Continuous volumetric representation
  * Uses ray integration
  * High quality but slow

* **Gaussian Splatting**:

  * Discrete explicit representation
  * Uses projected Gaussians
  * Real-time capable

* **Shared idea**:

  * Both rely on **alpha compositing**
  * Both approximate the same radiance field physics

---

# Related Notebooks:
* 1) gaussian splat : https://colab.research.google.com/drive/12X5Bj1Vq9yPV83cpQGGV_BuGQHWb1MCT?usp=sharing
* 2) nerf : https://colab.research.google.com/drive/1WiyZYPVM6SoA6K15AZM9FMM7pDOEniM2?usp=sharing
* 3) LOD streaming : https://colab.research.google.com/drive/1Tu5OqkdLLe_i3aNU6C87EHtrTD8rMafb?usp=sharing
