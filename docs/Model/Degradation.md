## Degradation of macroalgae detritus

This section describes the routine **p4zpocmac.F90** the computation of the degradation of macroalgal-derived particulate organic carbon (`POCMAC`) into dissolved organic carbon (`DOCMAC`) within the PISCES biogeochemical model. The transformation is temperature-dependent and represents a first-order remineralization process.

---
### Overview

At each time step, the degradation of `POCMAC` is computed based on the local seawater temperature. A proportion of the labile fraction (95%) of detrital macroalgae carbon is remineralized into DOC. The rate is calculated and applied to the tracer arrays, ensuring mass conservation in the organic matter cycle.

---
### Equation

The degradation rate of `POCMAC` is faster at higher temperature, following Filbee-Dexter et al (2022) :

$$
R_{remin,ijk} = (0.054 × T_ijk + 0.3605) × Δt
$$
$$
\frac{dPOCMAC}{dt} = - 0.95 × Remin_rate × POCMAC
$$

$$
\frac{dDOCMAC}{dt} = + 0.95 × Remin_rate × POCMAC
$$


Where:

- Only 95% of `POCMAC` is considered labile and degradable,
- The remaining 5% is refractory and assumed not to remineralize on model timescales.

---

### Variable Mapping

| Fortran Name                  | Description                                                       | Symbol                      |
|------------------------------|-------------------------------------------------------------------|-----------------------------|
| `ts(:,:,:,jp_tem,Kmm)`       | Temperature field at current time step                            | `T_ijk`                     |
| `xstep`                      | Duration of the tracer time step (in seconds)                     | `Δt`                        |
| `reminratepocmac`            | Temperature-dependent remineralization rate of POCMAC             | `Remin_rate_ijk`            |
| `iom_put("xdegradpoc", …)`   | Output variable for diagnostics                                   | –                           |



### Diagnostic Output

The routine outputs a 3D diagnostic field `xdegradpoc`, which represents the degradation flux (in mmol C m⁻³ d⁻¹) of particulate macroalgal carbon to DOC. This field is useful for analyzing the spatial pattern of macroalgae remineralization in the ocean.


---

### Reference 

Filbee-Dexter, K., Pessarrodona, A., Pedersen, M.F. et al. Carbon export from seaweed forests to deep ocean sinks. Nat. Geosci. 17, 552–559 (2024). https://doi.org/10.1038/s41561-024-01449-7

