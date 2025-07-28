# Sedimentation Routines – Macroalgae Module

This routine describes **p4zsed.F90** which handles the remineralization of particulate organic carbon from macroalgae (POCMAC) reaching the seafloor and its impacts on biogeochemical tracers such as nutrients, oxygen, carbon, and alkalinity.

## Description

The routine computes the remineralization fluxes from POC using a combination of:

* **Denitrification (NO₃ to NH₄)** under nitrate-rich, oxygen-depleted conditions.
* **Oxygen consumption** when O₂ is available.
* **Release of dissolved organic carbon (DOC)** and remineralized nutrients (PO₄³⁻, NH₄⁺).
* **Changes in total alkalinity (TAlk)** and **dissolved inorganic carbon (DIC)**.
* **Iron remineralization**, specifically from macroalgal origin.

Burial is applied to macroalgae-derived POC via `zbureffmac`.

---

## Equations

*  \(w_{\text{POC}} = \frac{F_{\text{POC}}}{h} \): local volumetric remineralization rate from sinking POC.
*  \(x_{\text{macro}} = \frac{F_{\text{POC,macro}}}{F_{\text{POC,tot}}}\): fraction of macroalgae POC.
*  \(R_{\text{stoich}}\): stoichiometric ratios applied to convert organic matter to remineralized nutrients.

Then:

1. **Total volumetric remineralization**:

   $$
   w_{\text{POC}} = \frac{F_{\text{POC,bio}} + F_{\text{POC,macro}}}{h}
   $$

2. **Denitrification flux (NO₃ → NH₄)**:

   $$
   P_{\text{denit}} = \min\left(0.5 \cdot \frac{[\text{NO}_3] - r_{\text{trn}}}{r_{\text{denit}}}, z_{\text{denit}}^{2D} \cdot w_{\text{POC}} \cdot R_{\text{rivNO3}}\right)
   $$

3. **Remaining remineralization not using NO₃**:

   $$
   R_{\text{O}_2,\text{limit}} = \min\left(\frac{[\text{O}_2] - r_{\text{trn}}}{r_{\text{O}_2}}, \left(w_{\text{POC}} \cdot R_{\text{rivNO3}} - P_{\text{denit}}\right) \cdot (1 - f_{\text{nitrif}})\right)
   $$

4. **Stoichiometric redistribution**:

    - **DIC**:
     $$
     \Delta \text{DIC} = P_{\text{denit}} + R_{\text{O}_2,\text{limit}}
     $$
   - **NH₄**:
     $$
     \Delta \text{NH}_4 = \left(P_{\text{denit}} + R_{\text{O}_2,\text{limit}}\right) \cdot \left[(1 - x_{\text{macro}}) + x_{\text{macro}} \cdot \frac{q_{C:N}^{\text{phy}}}{q_{C:N}^{\text{mac}}}\right]
     $$
   - **NO₃**:
     $$
     \Delta \text{NO}_3 = - r_{\text{denit}} \cdot P_{\text{denit}} \cdot \left[(1 - x_{\text{macro}}) + x_{\text{macro}} \cdot \frac{q_{C:N}^{\text{phy}}}{q_{C:N}^{\text{mac}}}\right]
     $$
   - **PO₄**:
     $$
     \Delta \text{PO}_4 = \left(P_{\text{denit}} + R_{\text{O}_2,\text{limit}}\right) \cdot \left[(1 - x_{\text{macro}}) + x_{\text{macro}} \cdot \frac{q_{C:P}^{\text{phy}}}{q_{C:P}^{\text{mac}}}\right]
     $$
   - **Fe**:
     $$
     \Delta \text{Fe} = \left(P_{\text{denit}} + R_{\text{O}_2,\text{limit}}\right) \cdot x_{\text{macro}} \cdot \frac{1}{q_{C:Fe}^{\text{mac}}}
     $$

Where macroalgae stoichiometry is mixed based on \(x_{\text{macro}}\), using:

* \(q_{C:N}^{\text{phy}}\), \(q_{C:P}^{\text{phy}}\) for phytoplankton
* \(q_{C:N}^{\text{mac}}\), \(q_{C:P}^{\text{mac}}\), \(q_{C:Fe}^{\text{mac}}\) for macroalgae.

---

## Variable Summary

| Variable             | Description                                                              | Units            |
| -------------------- | ------------------------------------------------------------------------ | ---------------- |
| `sinkpocb`           | Sinking flux of background POC                                           | mmol C m⁻² d⁻¹   |
| `sinkpocmac`         | Sinking flux of macroalgae-derived POC                                   | mmol C m⁻² d⁻¹   |
| `tr(:,:,jpno3)`      | Nitrate concentration                                                    | mmol N m⁻³       |
| `tr(:,:,jpoxy)`      | Oxygen concentration                                                     | mmol O₂ m⁻³      |
| `tr(:,:,jpdoc)`      | DOC concentration                                                        | mmol C m⁻³       |
| `tr(:,:,jpnh4)`      | Ammonium concentration                                                   | mmol N m⁻³       |
| `tr(:,:,jppo4)`      | Phosphate concentration                                                  | mmol P m⁻³       |
| `tr(:,:,jpdic)`      | Dissolved inorganic carbon concentration                                 | mmol C m⁻³       |
| `tr(:,:,jptal)`      | Total alkalinity                                                         | µmol eq m⁻³      |
| `tr(:,:,jpfer)`      | Iron concentration                                                       | µmol Fe m⁻³      |
| `qCN_phy`, `qCP_phy` | C\:N and C\:P ratios for phytoplankton                                   | mol\:mol         |
| `qCN_mac`, `qCP_mac` | C\:N and C\:P ratios for macroalgae                                      | mol\:mol         |
| `qCFe_mac`           | C\:Fe ratio for macroalgae                                               | mol\:mol         |
| `zbureffmac`         | Macroalgae burial efficiency                                             | dimensionless    |

---

