---
title: Calculating Hyperfine Coupling Constants in VASP
date: 2025-04-13
categories: [Computational Materials Science]
tags: [Defect, VASP]     # TAG names should always be lowercase
author: Shibu
description: Tutorial on Calculation of Hyperfine Coupling Constants
toc: true # default is true
pin: false
comments: true
math: true
---

## Decoding Defect Spins: Calculating Hyperfine Coupling Constants in VASP

Point defects, those tiny imperfections in crystalline lattices, often possess unpaired electron spins. These spins don't exist in isolation; they interact with the magnetic moments of nearby atomic nuclei. This interaction, known as **hyperfine coupling (HFC)**, acts like a unique fingerprint, revealing crucial details about the defect's identity, local environment, and electronic structure.

Techniques like Electron Paramagnetic Resonance (EPR), Electron Nuclear Double Resonance (ENDOR), and Optically Detected Magnetic Resonance (ODMR) directly measure hyperfine couplings. Comparing these experimental results with theoretical calculations provides a powerful way to identify defect structures and validate our understanding of their quantum mechanical nature.

Fortunately, VASP, a workhorse for first-principles calculations, can compute these hyperfine parameters, bridging the gap between theory and experiment. Here’s a guide on how to approach these calculations and what to keep in mind.

**What is Hyperfine Coupling?**

At its heart, hyperfine coupling describes the interaction energy between an electron's spin (**S**) and a nucleus's spin (**I**). This interaction is typically represented by a tensor **A**, leading to an interaction Hamiltonian:

$$H_{HFC} = S · A · I$$

The hyperfine tensor **A** contains the information we want to calculate and compare with experiments. It has two main contributions:

1.  **Fermi Contact Interaction (Isotropic):** This arises from the non-zero probability of finding the unpaired electron *at the position of the nucleus*. It's proportional to the electron spin density $$\|\psi(0)\|^2$$ right at the nucleus (r=0). This term is isotropic (scalar) and only exists for electrons with s-character at the nucleus (since only s-orbitals have non-zero density at r=0).
2.  **Dipolar Interaction (Anisotropic):** This is the classical magnetic dipole-dipole interaction between the electron's magnetic moment (distributed around the nucleus) and the nuclear magnetic moment. It depends on the distance and orientation between the electron spin distribution and the nuclear spin. This term is anisotropic (traceless tensor) and reflects the shape of the spin density distribution around the nucleus.

The total hyperfine tensor **A** calculated by VASP is the sum of these two contributions.

**Why Calculate HFC for Defects?**

*   **Defect Identification:** Comparing the calculated hyperfine tensor (principal values and axes) for different candidate defect structures with experimental EPR/ENDOR data is often the most definitive way to identify the microscopic nature of a measured defect signal.
*   **Electronic Structure Validation:** The HFC parameters are extremely sensitive to the spatial distribution of the unpaired electron's spin density. Calculating them validates whether the chosen computational method (functional, basis set/cutoff) accurately captures the defect's wavefunction localization and character.
*   **Understanding Bonding:** The HFC tensor reveals which nuclei the unpaired electron significantly interacts with, providing insights into the chemical bonding and hybridization around the defect.
*   **Predicting Spectra:** Theoretical calculations can predict HFC parameters for defects that haven't been experimentally characterized yet, guiding future experiments.

**Calculating HFC Parameters in VASP**

The good news is that VASP makes the calculation relatively straightforward, provided the underlying electronic structure calculation is sound.

1.  **Spin-Polarized Calculation:** Hyperfine coupling originates from unpaired electron spins. Therefore, you *must* perform a spin-polarized calculation by setting `ISPIN = 2` in the `INCAR` file.
2.  **The Magic Tag: `LHYPERFINE`:** To activate the calculation of the hyperfine tensor, simply add the following tag to your `INCAR`:
    `LHYPERFINE = .TRUE.`
3.  **PAW Potential Requirement:** VASP uses the Projector Augmented Wave (PAW) method. The crucial Fermi contact term depends on the electron density *inside* the PAW augmentation sphere (i.e., near the nucleus). Therefore, you need PAW potentials that accurately represent the all-electron density and spin density in the core region.
    *   Standard PAW potentials usually work, but those generated specifically for GW calculations (`_GW` suffix) are often recommended as they are typically generated with stricter criteria for core representation. Always check the `POTCAR` generation details if possible.
4.  **Run VASP:** Perform a standard VASP calculation (e.g., static calculation after geometry optimization).

**Output: Finding the Results**

The calculated hyperfine parameters are printed in the `OUTCAR` file. Search for the section starting with:

`Fermi contact (isotropic) hyperfine coupling parameter (MHz)`, `Dipolar hyperfine coupling parameters (MHz)`, `Total hyperfine coupling parameters after diagonalization (MHz)`

VASP typically outputs the following for each atom (identified by `NION`):

*   **Isotropic Fermi contact term (MHz):** This is the scalar part `A_iso`.
*   **Anisotropic dipolar tensor (MHz):** A 3x3 matrix representing the traceless dipolar contribution `A_dip`.
*   **Total hyperfine tensor (MHz):** The sum `A = A_iso * I + A_dip`, where `I` is the identity matrix.

The output usually includes the eigenvalues (principal values) and eigenvectors (principal axes) of the total tensor **A**.

**Crucial Considerations for Reliable HFC Calculations:**

Getting *accurate* HFC parameters requires more care than standard total energy calculations:

1.  **Accurate Ground State is Paramount:** HFC is highly sensitive to the spin density distribution. Garbage in, garbage out. Ensure your ground-state calculation is well-converged:
    *   **Geometry:** Use a tightly converged geometry (`EDIFFG` small, e.g., `-0.01`). Use `ISIF = 2` for defect relaxation in a fixed supercell.
    *   **Supercell Size:** Spin density localization can be affected by interactions with periodic images. Test convergence with respect to supercell size.
    *   **K-points:** Converge the total energy and, more importantly, the *magnetization* or spin density distribution with respect to the k-point mesh.
    *   **ENCUT:** Use a sufficiently high energy cutoff, converged for your system and PAW potentials.
2.  **Functional Choice Matters (A Lot!):** Standard GGA functionals (like PBE) are known to often over-delocalize electron density and spin density due to self-interaction errors. This can lead to significant underestimation of the Fermi contact term and inaccurate dipolar terms.
    *   **Hybrid Functionals (HSE06):** These functionals mix a portion of exact Hartree-Fock exchange, which partially corrects self-interaction error and generally leads to better localization of defect states and spin density. **Using HSE06 (`LHFCALC = .TRUE.`) is strongly recommended for quantitative HFC calculations.** Be aware of the increased computational cost.
    *   **DFT+U:** If your defect involves strongly correlated d or f electrons, DFT+U might be necessary to get the correct ground state spin configuration, but its effect on HFC needs careful validation.
3.  **PAW Potential Choice:** As mentioned, use PAW potentials that accurately describe the core region. Check `POTCAR` details or prefer `_GW` potentials if available and tested.
4.  **Correct Spin State:** Ensure your calculation converges to the experimentally relevant spin state (e.g., doublet S=1/2, triplet S=1). You might need to use the `NUPDOWN` tag to constrain the total spin magnetic moment if multiple spin states are close in energy. Check the converged total `magnetization` in the `OUTCAR`.
5.  **Convergence Thresholds:** Use tight electronic convergence criteria (e.g., `EDIFF = 1E-6` or smaller) as the spin density can be sensitive.
6.  **Relativistic Effects:** For heavy elements, spin-orbit coupling (`LSORBIT = .TRUE.`) can influence the spin density distribution and HFC parameters. This significantly increases computational cost and complexity (requiring non-collinear calculations, `LNONCOLLINEAR = .TRUE.`). For light elements, it's usually a smaller effect.

**Interpreting the Results:**

*   Compare the calculated principal values of the hyperfine tensor **A** with experimental data (often reported in MHz or Gauss). Remember to check units.
*   Analyze the orientation of the principal axes relative to the defect structure.
*   The magnitude of the Fermi contact term tells you about the s-character of the spin density at the nucleus.
*   The anisotropy (difference between principal values) reflects the p, d, etc., character and shape of the spin density distribution.

**Conclusion:**

Calculating hyperfine coupling parameters using VASP (`LHYPERFINE = .TRUE.`) provides a powerful link between first-principles theory and magnetic resonance experiments for point defects. While activating the calculation is simple, achieving accurate and reliable results demands careful attention to the underlying electronic structure calculation, particularly concerning spin polarization, functional choice (hybrid functionals highly recommended), PAW potentials, and convergence. When done carefully, HFC calculations offer invaluable insights into the microscopic world of defects in materials.

### Further Reading & References:

Calculating hyperfine coupling parameters bridges quantum mechanical theory with detailed experimental spectroscopy. The following references provide background on the theory, the computational methods within VASP, and examples of applications:

**Fundamental Concepts & PAW Implementation:**

1.  **Projector augmented-wave method**
    *   P. E. Blöchl
    *   *Physical Review B* **50**, 17953 (1994)
    *   *Comment:* The foundational paper describing the Projector Augmented Wave (PAW) method, which is essential for accurately calculating properties like the Fermi contact interaction that depend on the wavefunction near the nucleus.

2.  **From ultrasoft pseudopotentials to the projector augmented-wave method**
    *   G. Kresse and D. Joubert
    *   *Physical Review B* **59**, 1758 (1999)
    *   *Comment:* Describes the specific implementation of the PAW method used in VASP. Essential citation for VASP calculations.

3.  **Calculation of electron-positron momentum densities using the projector augmented-wave method**
    *   Z. Szotek, W. M. Temmerman, A. Svane, L. Petit, G. M. Stocks, and H. Winter
    *   *Physical Review B* **68**, 054415 (2003)
    *   *Comment:* While focused on positron annihilation, this paper discusses the calculation of electron densities within the PAW spheres, which is directly relevant to the Fermi contact term calculation in HFC. It provides insight into how core-region properties are handled.

**Defect Calculation Methodology (Context):**

4.  **First-principles calculations for defects and impurities: Applications to III-nitrides**
    *   C. G. Van de Walle and J. Neugebauer
    *   *Journal of Applied Physics* **95**, 3851 (2004)
    *   *Comment:* Foundational review on defect calculations, covering aspects like supercells, convergence, and the importance of functionals, which are crucial prerequisites for reliable HFC calculations.

5.  **First-principles calculations of point defects in solids**
    *   C. Freysoldt, B. Grabowski, T. Hickel, J. Neugebauer, G. Kresse, A. Janotti, and C. G. Van de Walle
    *   *Reviews of Modern Physics* **86**, 253 (2014)
    *   *Comment:* More recent comprehensive review, including discussions on charged states and advanced functional use, relevant for setting up the correct ground state before calculating HFC.

**Importance of Functionals & Application Examples:**

6.  **Hyperfine interactions of the silicon vacancy in 4H- and 6H-SiC from density functional theory**
    *   K. Szász, T. Hornos, M. Marsman, and A. Gali
    *   *Physical Review B* **88**, 075202 (2013)
    *   *Comment:* A specific example comparing different functionals (including GGA and HSE hybrids) for calculating HFC parameters of a well-known defect (silicon vacancy) and comparing them to experimental data. Clearly demonstrates the improved accuracy often obtained with hybrid functionals.

7.  **Quantum computing with defects**
    *   J. R. Weber, W. F. Koehl, J. B. Varley, A. Janotti, B. B. Buckley, C. G. Van de Walle, and D. D. Awschalom
    *   *Proceedings of the National Academy of Sciences* **107**, 8513 (2010)
    *   *Comment:* While focused on quantum computing applications, this paper involves detailed first-principles calculations (including HFC) of defects like the nitrogen-vacancy (NV) center in diamond, showcasing the use of DFT to understand experimentally relevant parameters.

8.  **First-principles theory of nonradiative carrier capture at point defects**
    *   A. Alkauskas, Q. Yan, and C. G. Van de Walle
    *   *Physical Review B* **90**, 075202 (2014)
    *   *Comment:* Although primarily about carrier capture, this paper details careful methodology for defect calculations, including the use of hybrid functionals and considerations for spin states, which are directly applicable to setting up accurate HFC calculations.

**Classic Texts on Hyperfine Interactions (Theoretical Background):**

9.  **Principles of Magnetic Resonance**
    *   C. P. Slichter
    *   *Springer Series in Solid-State Sciences* (3rd Edition or later)
    *   *Comment:* A classic textbook providing deep theoretical background on magnetic resonance phenomena, including detailed derivations and explanations of the hyperfine interaction (Fermi contact and dipolar terms).

10. **Electron Paramagnetic Resonance of Transition Ions**
    *   A. Abragam and B. Bleaney
    *   *Oxford Classic Texts in the Physical Sciences* (Reprint Edition)
    *   *Comment:* Another seminal textbook focused on EPR, containing extensive theoretical treatments of the spin Hamiltonian, including hyperfine structure, for transition metal ions and defects.

**VASP Manual:**

11. **The VASP Manual (online or PDF)**
    *   Available from the VASP website (requires license).
    *   *Comment:* Essential practical guide. Check the sections on `LHFCALC`, `ISPIN`, `LNONCOLLINEAR`, `LSORBIT`, and potentially discussions on PAW potentials (`POTCAR`) for specific tag information and context.

12. [Calculating the hyperfine coupling constant](https://www.vasp.at/wiki/index.php/Calculating_the_hyperfine_coupling_constant)
13. [LHYPERFINE](https://www.vasp.at/wiki/index.php/LHYPERFINE)
14. [NGYROMAG](https://www.vasp.at/wiki/index.php/NGYROMAG)
