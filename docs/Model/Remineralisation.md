# Remineralization of Macroalgal DOC 

This is a description of **p4zrem.F90**.
To incorporate the remineralization of dissolved organic carbon from macroalgal origin (`DOCMAC`) into the biogeochemical model, we extended the existing remineralization scheme—originally limited to phytoplankton-derived DOC. This document summarizes the added formulations and their stoichiometric interpretation.

---

## 1. Fraction of Macroalgal DOC

We first calculate the fraction of total DOC originating from macroalgae:

$$ f_{DOCmac} = \frac{DOC_{mac}}{DOC + DOC_{mac} + \varepsilon} $$

Here,  \( \varepsilon \)  is a small number to prevent division by zero.

---

## 2. Remineralization Fluxes

Oxic, suboxic and anoxic remineralization are computing as in PISCES std with `DOC` + `DOCMAC`.

---

## 3. Total Remineralization Flux

The total remineralization flux is the sum of all pathways:

$$
R_{tot} =R_{oxic} + R_{denit} + R _{ano}
$$

---

## 4. DOC and DOC\_mac Evolution

Remineralization is split between the phytoplankton-derived DOC and macroalgal DOC pools using \( f_{DOCmac} \)  :

$$
\frac{dDOC}{dt} = -R_{tot} \cdot (1 - f_{DOCmac})
$$

$$
\frac{dDOC_{mac}}{dt} = -R_{tot} \cdot f_{DOCmac}
$$

---

## 5. Remineralization Products

### Dissolved Inorganic Carbon (DIC)

All remineralized DOC contributes to DIC:

$$
\frac{dDIC}{dt} = + R_{tot}
$$

### Oxygen

Only consumed in oxic remineralization:

$$
\frac{dO_2}{dt} = - R_{oxic} \cdot r_{O_2}
$$

### Nitrate and Ammonium

Stoichiometry accounts for distinct C:N ratios of phytoplankton and macroalgae:

$$
\frac{dNO_3^-}{dt} = - R_{denit} \cdot r_{denit} \cdot \left[(1 - f_{DOCmac}) + f_{DOCmac} \cdot \frac{Q_{C:N}^{PHY}}{Q_{C:N}^{MAC}}\right]
$$

$$
\frac{dNH_4^+}{dt} = + R_{tot} \cdot \left[(1 - f_{DOCmac}) + f_{DOCmac} \cdot \frac{Q_{C:N}^{PHY}}{Q_{C:N}^{MAC}}\right]
$$

### Total Alkalinity (TAl)

Alkalinity changes reflect nitrate consumption and denitrification:

$$
\frac{dTA}{dt} = + r_{NO_3} \cdot \left( R_{tot} + r_{denit} \cdot R_{denit} \right) \cdot \left[(1 - f_{DOCmac}) + f_{DOCmac} \cdot \frac{Q_{C:N}^{PHY}}{Q_{C:N}^{MAC}} \right]
$$

### Phosphate

Scaled similarly using C:P ratios:

$$
\frac{dPO_4^{3-}{dt} = + R_{tot} \cdot \left[(1 - f_{DOCmac}) + f_{DOCmac} \cdot \frac{Q_{C:P}^{PHY}}{Q_{C:P}^{MAC}} \right]
$$

### Iron (from macroalgae only)

Iron is released as dissolved iron only from macroalgal DOC remineralization:

$$
\frac{dDFe}{dt} = + R_{tot} \cdot f_{DOCmac} \cdot \left( \frac{1}{Q_{C:Fe}^{MAC}} \right)
$$

---

## Variables

| Fortran Variable       | Documentation / Equation Name           | Description                                                           |
|------------------------|-----------------------------------------|-----------------------------------------------------------------------|
| `xdocmacdoc`           | \( f_{\text{mac}} = \frac{\text{DOC}_{\text{MAC}}}{\text{DOC} + \text{DOC}_{\text{MAC}}} \) | Fraction of total DOC from macroalgae                                |
| `tr(:,:,:,jpdoc,...)`  | \( \text{DOC} \)                         | Classical dissolved organic carbon                                    |
| `tr(:,:,:,jpdocmac,...)` | \( \text{DOC}_{\text{MAC}} \)         | DOC from macroalgae                                                   |
| `tr(:,:,:,jpoxy,...)`  | \( \text{O}_2 \)                         | Dissolved oxygen                                                      |
| `tr(:,:,:,jpno3,...)`  | \( \text{NO}_3^- \)                      | Nitrate                                                               |
| `tr(:,:,:,jpdic,...)`  | \( \text{DIC} \)                         | Dissolved inorganic carbon                                            |
| `tr(:,:,:,jpnh4,...)`  | \( \text{NH}_4^+ \)                      | Ammonium                                                              |
| `tr(:,:,:,jptal,...)`  | \( \text{TA} \)                          | Total alkalinity                                                      |
| `tr(:,:,:,jppo4,...)`  | \( \text{PO}_4^{3-} \)                   | Phosphate                                                             |
| `tr(:,:,:,jpfer,...)`  | \( \text{DFe} \)                          | Dissolved iron                                                        |
| `qCN_phy`              | \( Q_{\text{C:N}}^{\text{phy}} \)       | Phytoplankton carbon-to-nitrogen ratio                               |
| `qCN_mac`              | \( Q_{\text{C:N}}^{\text{mac}} \)       | Macroalgae carbon-to-nitrogen ratio                                  |
| `qCP_phy`              | \( Q_{\text{C:P}}^{\text{phy}} \)       | Phytoplankton carbon-to-phosphorus ratio                             |
| `qCP_mac`              | \( Q_{\text{C:P}}^{\text{mac}} \)       | Macroalgae carbon-to-phosphorus ratio                                |
| `qCFe_mac`             | \( Q_{\text{C:Fe}}^{\text{mac}} \)      | Macroalgae carbon-to-iron ratio                                      |


