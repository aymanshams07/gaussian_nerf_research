# gaussian_nerf_research
research and dev work
Great — this is exactly the level Miris interviews will expect. I’ll go **clean, mathematical, and step-by-step**, focusing on intuition + derivation (not just definitions).

---

# 1) Deriving Neural Radiance Fields from Scratch

---

## Step 1: What are we modeling?

We want a function:

[
F(\mathbf{x}, \mathbf{d}) \rightarrow (c, \sigma)
]

* ( \mathbf{x} \in \mathbb{R}^3 ): 3D position
* ( \mathbf{d} \in \mathbb{R}^3 ): viewing direction
* ( c \in \mathbb{R}^3 ): RGB color
* ( \sigma \in \mathbb{R} ): density

---

## Step 2: Ray parameterization

A camera ray is:

[
\mathbf{r}(t) = \mathbf{o} + t\mathbf{d}
]

* ( \mathbf{o} ): camera origin
* ( \mathbf{d} ): direction
* ( t ): depth along ray

---

## Step 3: Volume Rendering Equation

Color of a pixel = integration along the ray:

[
C(\mathbf{r}) = \int_{t_n}^{t_f} T(t),\sigma(\mathbf{r}(t)),c(\mathbf{r}(t), \mathbf{d}),dt
]

Where:

* ( T(t) ): transmittance (probability ray reaches (t))

---

## Step 4: Transmittance derivation

[
T(t) = \exp\left(-\int_{t_n}^{t} \sigma(\mathbf{r}(s)) ds\right)
]

Interpretation:

* Accumulates how much density blocks light before point (t)

---

## Step 5: Discretization (critical for implementation)

Sample points along ray:

[
t_1, t_2, ..., t_N
]

Define:

[
\alpha_i = 1 - \exp(-\sigma_i \delta_i)
]

[
T_i = \prod_{j < i} (1 - \alpha_j)
]

Final color:

[
C = \sum_{i=1}^{N} T_i \alpha_i c_i
]

---

## Step 6: Neural Parameterization

Instead of storing the function explicitly:

[
F_\theta(\mathbf{x}, \mathbf{d}) = (\sigma, c)
]

* MLP learns scene
* Input uses positional encoding

---

## Key Insight

NeRF = **continuous volumetric density + rendering integral**

---

# 2) Deriving 3D Gaussian Splatting

---

## Step 1: Replace volume with discrete primitives

Instead of a continuous field:

We represent scene as **N Gaussians**:

Each Gaussian:

[
G_i(\mathbf{x}) = w_i \cdot \mathcal{N}(\mathbf{x} \mid \mu_i, \Sigma_i)
]

Where:

* ( \mu_i ): center
* ( \Sigma_i ): covariance (shape/orientation)
* ( w_i ): opacity/intensity

---

## Step 2: Gaussian density definition

[
\mathcal{N}(\mathbf{x}) = \frac{1}{(2\pi)^{3/2} |\Sigma|^{1/2}} \exp\left(-\frac{1}{2}(\mathbf{x}-\mu)^T \Sigma^{-1} (\mathbf{x}-\mu)\right)
]

---

## Step 3: Projection to screen (critical step)

We project 3D Gaussian → 2D screen Gaussian.

Using camera projection:

[
\mathbf{x}_{screen} = \Pi(\mathbf{x})
]

Then covariance transforms:

[
\Sigma_{2D} = J \Sigma_{3D} J^T
]

* (J): Jacobian of projection

---

## Step 4: Rendering (alpha compositing)

Sort Gaussians front-to-back.

For each pixel:

[
C = \sum_i T_i \alpha_i c_i
]

Same structure as NeRF!

Where:

* ( \alpha_i ): opacity of projected Gaussian
* ( T_i = \prod_{j<i}(1-\alpha_j) )

---

## Step 5: Why it works

Gaussian splatting ≈ **discretized radiance field**

Instead of:

* Sampling along ray (NeRF)

We:

* Directly rasterize splats

---

## Key Insight

NeRF → continuous integral
Gaussian splatting → **explicit, rasterizable approximation**

---

# 3) Plane-Based Question (Interview Style Derivation)

---

## Problem

**Given a 3D Gaussian, derive its projection onto a plane (camera image plane).**

---

## Step 1: Plane equation

A plane:

[
\mathbf{n}^T \mathbf{x} + d = 0
]

---

## Step 2: Local linearization of projection

Camera projection:

[
\mathbf{x}_{2D} = \Pi(\mathbf{x})
]

Approximate near mean ( \mu ):

[
\Pi(\mathbf{x}) \approx \Pi(\mu) + J(\mathbf{x}-\mu)
]

---

## Step 3: Transform Gaussian

Original:

[
\mathbf{x} \sim \mathcal{N}(\mu, \Sigma)
]

After linear transform:

[
\mathbf{y} = J\mathbf{x}
]

Then:

[
\mathbf{y} \sim \mathcal{N}(J\mu, J\Sigma J^T)
]

---

## Step 4: Restrict to plane

Projection already maps to 2D plane:

So resulting distribution:

[
\mathcal{N}(\mu_{2D}, \Sigma_{2D})
]

Where:

* ( \mu_{2D} = \Pi(\mu) )
* ( \Sigma_{2D} = J \Sigma J^T )

---

## Step 5: Geometric interpretation

* 3D ellipsoid → 2D ellipse
* Orientation preserved via covariance
* Size depends on depth + perspective

---

## Step 6: Why this matters (Miris-level insight)

This step is the **core of real-time rendering in Gaussian splatting**:

* Avoid ray marching
* Directly rasterize ellipses
* Enables GPU acceleration

---

# Final Big Picture (What Interviewers Want)

You should be able to say:

* NeRF = **integral along rays of density**
* Gaussian splatting = **projected anisotropic kernels**
* Both share:

  * Alpha compositing
  * Radiance accumulation
* Key difference:

  * NeRF → implicit + slow
  * Splatting → explicit + real-time



