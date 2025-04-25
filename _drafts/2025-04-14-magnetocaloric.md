---
title: Untangling Magnetism
date: 2025-04-14
categories: [Computational Materials Science]
tags: [Magnetism, VASP]     # TAG names should always be lowercase
author: Shibu
description: Tutorial on Calculating M(T, B) with VASP
toc: true # default is true
pin: false
comments: true
math: true
---

## Untangling Magnetism: Calculating M(T, B) with VASP

Magnetism is one of the most fascinating properties of materials, driving technologies from data storage to energy conversion. A fundamental characteristic is the magnetization (M), especially how it behaves under varying temperature (T) and external magnetic field (B). If you're using the Vienna Ab initio Simulation Package (VASP) for materials modeling, you might wonder: "How can I calculate the M(T, B) curve?"

The short answer? VASP, primarily a ground-state (0 Kelvin) Density Functional Theory (DFT) code, **doesn't directly calculate the full thermodynamic M(T, B) curve on its own.** Finite temperature magnetic behavior, particularly phase transitions like the Curie temperature (Tc), arises from the thermal fluctuations of spins governed by statistical mechanics. Standard DFT doesn't inherently capture this entropy-driven disordering.

But don't despair! VASP is a crucial *first step* in a multi-stage process. You use VASP to calculate the fundamental magnetic interactions at 0 K, and then feed these parameters into statistical mechanics models. Let's break down the workflow.

### Step 1: Ground State Magnetism with VASP (T=0 K)

This is where VASP shines. You can calculate the magnetization under different conditions at absolute zero.

**A) Zero External Field (B=0):**

1.  **Setup:** Perform a spin-polarized VASP calculation (`ISPIN = 2` in the INCAR).
2.  **Initial Moments:** Provide initial magnetic moments for the magnetic atoms using the `MAGMOM` tag. For example, `MAGMOM = 2*5.0` for ferromagnetic iron, or `MAGMOM = 5.0 -5.0` for a simple antiferromagnetic setup. VASP will relax these moments self-consistently.
3.  **Output:** After convergence, check the OUTCAR file:
    *   Search for `magnetization (x)`, `magnetization (y)`, `magnetization (z)` near the end for the total magnetic moment vector of the simulation cell (in $$μ_B$$ per cell).
    *   The `magnetization` value on the `TOTEN` line also gives the total moment.
    *   For site-projected moments (useful!), set `LORBIT = 10` or `11` in the INCAR and check the values listed per atom in the OUTCAR.
4.  **Result:** This gives you the ground-state magnetization M(T=0, B=0). For ferromagnets, this is the saturation magnetization. For antiferromagnets, it should ideally be zero (though you'll see non-zero site-projected moments).

**B) With External Field (B≠0) at T=0 K:**

VASP *can* include an external magnetic field, which primarily couples to the orbital magnetic moment via the Zeeman effect.

1.  **Setup:** In addition to `ISPIN = 2` to your INCAR.
2.  **Field Strength:** Specify the field using the `BEXT` tag (in eV). Example: `BEXT = 1.0` in collinear calculation and `BEXT = 1.0 0.0 0.0` noncollinear calculations applies a $$B_0$$ T magnetic field along the x-direction. Here, $$B_0 = \frac{2B_{ext}}{g_e \mu_B}$$, where $$\mu_B= 5.788 381 8060E-5~eV~T^{-1}$$, and $$g_e= 2.002 319 304 362 56$$.
3.  **Output:** Check the magnetization in the OUTCAR as described above.
4.  **Result:** This yields M(T=0, B). It tells you how the ground state electronic structure and moments respond to the field *without* considering thermal fluctuations.

### Step 2: Extracting Magnetic Interaction Parameters using VASP

This is the critical link between VASP and finite-temperature behavior. We need to parameterize an effective spin Hamiltonian, most commonly the Heisenberg model:

$$H = - \sum_{ij}{J_{ij} S_i ⋅ S_j} - \sum_{i}{K_x (S_i^x)^2} - \sum_{i}{K_y (S_i^y)^2} - \sum_{i}{K_z (S_i^z)^2} - μ_B \sum_{i}{g_i B ⋅ S_i}$$

Here:
*   $$S_i$$: Spin vector at magnetic site `i`.
*   $$J_{ij}$$: Exchange interaction parameter between sites `i` and `j`. Positive `J` favors ferromagnetism (parallel alignment), negative `J` favors antiferromagnetism (anti-parallel).
*   $$K_i$$: Magnetic Anisotropy Energy (MAE) constant (optional but often important). Describes the preferred orientation of spins.
*   $$B$$: External magnetic field.
*   $$g_i$$: g-factor (often approximated as 2).
*   $$μ_B$$: Bohr magneton.

**How to get the `Jᵢⱼ` (and `Kᵢ`) parameters from VASP:**

1.  **Energy Mapping:**
    *   **Calculate Total Energies:** Perform highly converged VASP calculations (`ISPIN=2`) for different magnetic configurations in suitable supercells. Examples:
        *   Ferromagnetic (FM) state.
        *   Various Antiferromagnetic (AFM) states (e.g., A-type, C-type, G-type, depending on the lattice).
        *   Maybe non-collinear states if needed.
    *   **Map Energies to Hamiltonian:** Relate the energy *differences* between these configurations to sums of the `Jᵢⱼ` parameters based on the Heisenberg model. For example, `E_AFM - E_FM` can often be expressed in terms of nearest-neighbor and next-nearest-neighbor `J` values. Solving a system of equations from multiple configurations allows you to extract individual `Jᵢⱼ` values.
    *   **Anisotropy (Kᵢ):** Calculate the energy difference when the magnetization (in the FM state) is aligned along different crystallographic axes (e.g., `E_z - E_x`). This requires `LSORBIT = .TRUE.` and careful setup of `SAXIS` and initial `MAGMOM`.

2.  **Alternative Methods (More advanced):**
    *   **Liechtenstein-Katsnelson-Antropov-Gubanov (LKAG) method:** Calculates `Jᵢⱼ` using Green's functions from a single reference state (often FM). Can sometimes be implemented within or alongside DFT codes.
    *   **Spin Spiral Calculations:** Use VASP's non-collinear capabilities (`LNONCOLLINEAR = .TRUE.`) to calculate the energy `E(q)` of spin spirals. The Fourier transform of `J(q)` (related to `E(q)`) gives the real-space `Jᵢⱼ`.

**Key VASP Inputs for this step:** `ISPIN=2`, `MAGMOM` (crucial!), `EDIFF` (needs high precision), potentially `LNONCOLLINEAR`, `LSORBIT`, `SAXIS` depending on the method and system.

### Step 3: Statistical Mechanics Simulations (Calculating M(T, B))

Now, armed with your `Jᵢⱼ`, `Kᵢ`, and the magnitude of the spins `|Sᵢ|` (often taken from the T=0 VASP calculation), you leave VASP behind and use a statistical mechanics code.

1.  **Method:** The most common method is **Classical Monte Carlo (MC)** simulation based on the Metropolis algorithm.
    *   **Setup:** Create a simulation box (typically a large supercell) containing classical spin vectors `Sᵢ` on each magnetic site.
    *   **Simulation:**
        *   Initialize spins randomly or in an ordered state.
        *   Iteratively propose random changes to spin orientations.
        *   Accept or reject changes based on the energy change (calculated using the spin Hamiltonian `H` including the `B⋅Sᵢ` term) and the temperature `T` via the Boltzmann factor `exp(-ΔE / k<0xE2><0x82><0x9_B>T)`.
        *   Run the simulation for many steps to reach thermal equilibrium at a specific (T, B) point.
        *   Calculate the average total magnetization `<M> = <Σᵢ gᵢ μ<0xC2><0xB_B> Sᵢ>` over many equilibrium configurations.
    *   **Repeat:** Perform simulations across a range of desired temperatures and magnetic fields to map out the M(T, B) behavior, including identifying phase transitions like Tc (where M drops significantly).

2.  **Software:** You'll need external software for this step. Popular choices include:
    *   **Vampire:** A dedicated atomistic spin dynamics/Monte Carlo code.
    *   **UppASD:** Another powerful code for spin dynamics and MC.
    *   **MuSpin:** A GPU-accelerated MC code.
    *   **Custom Scripts:** Many researchers write their own MC codes (e.g., in Python with libraries like NumPy).

3.  **Alternative Statistical Methods:**
    *   **Spin Dynamics (SD):** Solves the Landau-Lifshitz-Gilbert equation. More computationally intensive for static M(T) but provides dynamic info.
    *   **Mean Field Theory (MFT) / Random Phase Approximation (RPA):** Analytical approximations based on `Jᵢⱼ`. Computationally cheap but less accurate, especially near Tc.

### What About Ab Initio Molecular Dynamics (AIMD)?

You *can* run VASP MD (`IBRION = 0`) at finite temperature (`TEBEG`) with `ISPIN = 2`. However, **this is generally NOT the correct way to get thermodynamic M(T) curves:**

*   The electronic temperature `TEBEG` mainly affects Fermi-Dirac smearing, not the collective spin fluctuations (magnons) that drive thermal demagnetization.
*   AIMD struggles to capture the entropy-driven disorder of the paramagnetic state above Tc.
*   It's computationally extremely expensive and limited in system size and simulation time.

### Summary Workflow

1.  **VASP (T=0K):** Calculate ground state M(B=0) and potentially M(B≠0) using `ISPIN=2`, `MAGMOM`, `BEXT`.
2.  **VASP (Parameter Extraction):** Calculate total energies for various magnetic configurations (FM, AFM...).
3.  **Analysis:** Extract exchange parameters (`Jᵢⱼ`) and anisotropy (`Kᵢ`) by mapping VASP energies onto a spin Hamiltonian (e.g., Heisenberg).
4.  **External Code (Monte Carlo):** Use the extracted parameters (`Jᵢⱼ`, `Kᵢ`, `|Sᵢ|`) as input for a Monte Carlo simulation code.
5.  **MC Simulation:** Run simulations at various temperatures (T) and external fields (B) to compute the average magnetization `<M(T, B)>`.

### Conclusion

Calculating the full temperature and field-dependent magnetization M(T, B) from first principles is a powerful capability, but it requires moving beyond standard T=0 K DFT. By using VASP to accurately determine the fundamental magnetic interactions (`Jᵢⱼ`, `Kᵢ`) and then employing these parameters in statistical mechanics simulations (like Monte Carlo), you can effectively model and predict the rich magnetic behavior of materials across different conditions. It's a multi-step process, but one that bridges the quantum mechanical origins of magnetism with its macroscopic thermodynamic behavior.

**Have you used this approach or found alternative ways? Share your experiences in the comments!**

## References

**1. VASP Manual (Essential Reference for VASP Tags)**

*   **VASP Wiki:** [https://www.vasp.at/wiki/index.php/Main_Page](https://www.vasp.at/wiki/index.php/Main_Page)
    *   **`ISPIN`:** [https://www.vasp.at/wiki/index.php/ISPIN](https://www.vasp.at/wiki/index.php/ISPIN)
    *   **`MAGMOM`:** [https://www.vasp.at/wiki/index.php/MAGMOM](https://www.vasp.at/wiki/index.php/MAGMOM)
    *   **`LORBIT`:** [https://www.vasp.at/wiki/index.php/LORBIT](https://www.vasp.at/wiki/index.php/LORBIT)
    *   **`BEXT`:** [https://www.vasp.at/wiki/index.php/BEXT](https://www.vasp.at/wiki/index.php/BEXT)
    *   **`LSORBIT`:** [https://www.vasp.at/wiki/index.php/LSORBIT](https://www.vasp.at/wiki/index.php/LSORBIT) (for anisotropy)
    *   **`LNONCOLLINEAR`:** [https://www.vasp.at/wiki/index.php/LNONCOLLINEAR](https://www.vasp.at/wiki/index.php/LNONCOLLINEAR) (for non-collinear magnetism / spin spirals)
    *   **`SAXIS`:** [https://www.vasp.at/wiki/index.php/SAXIS](https://www.vasp.at/wiki/index.php/SAXIS) (for anisotropy / non-collinear direction)

    *Why these are relevant:* The official documentation explains the precise meaning and usage of the VASP input tags needed for magnetic calculations.

**2. Extracting Exchange Interactions (Jij)**

*   **Energy Mapping Approach (General Concept & Examples):**
    *   Xiang, H. J., Lee, C., & Whangbo, M. H. (2011). *Obtaining Reliable Exchange Parameters for Microscopic Magnetic Models from Density Functional Calculations*. In *Computational Methods for Large Systems* (pp. 235-264). John Wiley & Sons, Inc. (Book chapter providing a good overview).
    *   Pajda, M., Kudrnovský, J., Turek, I., Drchal, V., & Bruno, P. (2001). *Ab initio calculations of exchange interactions, spin-wave stiffness constants, and Curie temperatures of Fe, Co, and Ni*. Physical Review B, 64(17), 174402. DOI: `10.1103/PhysRevB.64.174402`
        *Why relevant:* Classic paper demonstrating the calculation of Jijs and subsequent Tc estimation for elemental ferromagnets using energy mapping (though often using Korringa-Kohn-Rostoker (KKR) method, the principle applies).

*   **Liechtenstein-Katsnelson-Antropov-Gubanov (LKAG) Method:**
    *   Liechtenstein, A. I., Katsnelson, M. I., Antropov, V. P., & Gubanov, V. A. (1987). *Local spin density functional approach to the theory of exchange interactions in ferromagnetic metals and alloys*. Journal of Magnetism and Magnetic Materials, 67(1), 65-74. DOI: `10.1016/0304-8853(87)90721-9`
        *Why relevant:* The original paper outlining the infinitesimal rotation method for calculating Jij based on Green's functions.

*   **Spin Spiral Method:**
    *   Kurz, P., Förster, F., Nordström, L., Bihlmayer, G., & Blügel, S. (2004). *Ab initio treatment of noncollinear magnets with the full-potential linearized augmented plane wave method*. Physical Review B, 69(2), 024415. DOI: `10.1103/PhysRevB.69.024415`
        *Why relevant:* Discusses the calculation of spin spiral states and extracting information, often used in the context of the FLAPW method but applicable concepts for VASP's non-collinear features.

**3. Statistical Mechanics Simulations (Monte Carlo / Spin Dynamics)**

*   **Textbooks (General Methods):**
    *   Newman, M. E. J., & Barkema, G. T. (1999). *Monte Carlo Methods in Statistical Physics*. Oxford University Press. (Classic textbook on MC methods).
    *   Landau, D. P., & Binder, K. (2014). *A Guide to Monte Carlo Simulations in Statistical Physics*. Cambridge University Press. (Comprehensive guide to MC simulations).
*   **Review Articles (Specific to Magnetism):**
    *   Evans, R. F. L., Hinzke, D., Atxitia, U., Nowak, U., Chantrell, R. W., & Chubykalo-Fesenko, O. (2014). *Atomistic spin model simulations of magnetic nanomaterials*. Journal of Physics: Condensed Matter, 26(10), 103202. DOI: `10.1088/0953-8984/26/10/103202`
        *Why relevant:* Reviews atomistic spin models (like Heisenberg) and simulation techniques (MC, SD) applied to magnetic materials.

**4. Simulation Software (Examples)**

*   **Vampire:** [https://vampire.york.ac.uk/](https://vampire.york.ac.uk/) (Atomistic Spin Dynamics and Monte Carlo)
    *   Evans, R.F.L., et al. (2014) J. Phys.: Condens. Matter 26 103202 (See above)
*   **UppASD:** [https://www.physics.uu.se/research/materials-theory/ongoing-research/uppasd/](https://www.physics.uu.se/research/materials-theory/ongoing-research/uppasd/) (Atomistic Spin Dynamics)
    *   Skubic, B., Hellsvik, J., Nordström, L., & Eriksson, O. (2008). *A method for atomistic spin dynamics simulations: implementation and examples*. Journal of Physics: Condensed Matter, 20(31), 315203. DOI: `10.1088/0953-8984/20/31/315203`
*   *(Note: Many researchers also use custom codes, especially for standard Metropolis MC).*

**5. Combined DFT + Statistical Mechanics Studies (Examples/Reviews)**

*   Antropov, V. P. (2003). *Challenges in the theory of magnetic materials*. Journal of Magnetism and Magnetic Materials, 262(2), L192-L197. DOI: `10.1016/S0304-8853(03)00206-3`
    *Why relevant:* Discusses bridging DFT and finite temperature magnetism.
*   Eriksson, O., Bergman, A., Bergqvist, L., & Hellsvik, J. (2017). *Atomistic Spin Dynamics: Foundations and Applications*. Oxford University Press. (Book covering the connection from DFT to spin dynamics).

