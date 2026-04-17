## SURFACE_COMPLEXATION

Surface complexation follows the approach outlined in Dzombak and Morel
(1990), with either a double layer or non-electrostatic model possible.
Currently, complexation must be on a specific mineral, so a valid
mineral name (listed in the **MINERALS** keyword block) must be given,
for example:

    >FeOH_strong on Fe(OH)3
    >FeOH_weak on Fe(OH)3

To specify a non-electrostatic model, the mineral name should be
followed by the keyword *--no_edl*, for example:

    >FeOH_strong on Fe(OH)3 -no_edl
    >FeOH_weak on Fe(OH)3   -no_edl

The capability for surface complexation on the bulk material will be
added soon.

### Notes on `SolidDensity` and surface complexation calculations

For surface complexation on a **specific mineral** (the currently
supported mode), the site inventory is computed from:

- site density (mol sites / m^2 mineral),
- specific surface area (m^2 / g mineral), and
- mineral abundance (from mineral moles or mineral volume fraction).

In other words, the core surface-site calculation is mineral-based and
does **not** directly use the `SolidDensity` input.

`SolidDensity` does still affect conversions for reported units such as
`mol/g solid` via the bulk solid:solution ratio, and it is also used by
bulk-material exchange calculations. When `SolidDensity` is not directly
prescribed (or is set to `CalculateFromMinerals`), the code can compute a
bulk solid density from mineral volume fractions together with mineral
molecular weights and molar volumes.

### Unit conversions used in printed surface-complexation output

The speciation/equilibrium output tables commonly print three related
quantities:

- `Sites/kgw` (or `Moles/kgw`)  
- `Mol/g solid` (or `Sites/g solid`)  
- `Mol/m^3 bulk` (or `Sites/BulkVolume(m^3)`)

The code builds a temporary bulk solid:solution ratio for these
conversions as:

$$
\mathrm{SolidSolutionRatioTemp} = 1000\cdot \rho_s \cdot (1-\phi)
$$

where $\rho_s$ is `SolidDensity` (kg/m$^3$) and $\phi$ is porosity.
Because $1\ \mathrm{kg}=1000\ \mathrm{g}$ and $(1-\phi)$ is the solid
volume fraction, this has units of grams of solid per cubic meter bulk
medium (numerically equivalent to g/L bulk).

From there, printed conversions are:

1.  **Sites/kgw $\rightarrow$ Sites/g solid**

    $$
    C_{\mathrm{g\ solid}}=\frac{C_{\mathrm{kgw}}}
    {\mathrm{SolidSolutionRatio}}
    $$

    (implemented as `.../SolidSolutionRatioTemp` in speciation output).

2.  **Sites/kgw $\rightarrow$ Sites/m$^3$ bulk**

    $$
    C_{\mathrm{bulk}} = C_{\mathrm{kgw}}\cdot \phi \cdot \rho_w
    $$

    where $\rho_w$ is fluid density (`rocond`, kg/m$^3$). This appears in
    equilibrium/speciation print routines as multiplication by
    `porcond*rocond`.

3.  **Inverse conversion (for context): Sites/m$^3$ bulk $\rightarrow$
    Sites/kgw**

    $$
    C_{\mathrm{kgw}}=\frac{C_{\mathrm{bulk}}}{\phi\cdot \rho_w}
    $$

    In the code this factor is often represented by `AqueousToBulk`.

If `SolidSolutionRatio` is zero, the code prints `0.0` in the
`Mol/g solid` column to avoid divide-by-zero.
