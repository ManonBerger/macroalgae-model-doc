# Sinking of Macroalgal Detritus

This documentation describes the routines **p4zsinkmac.F90** and **trcsink.F90**, which include sinking of macroalgae detritus `POCMAC`.

---

## Overview

In this module, the sinking of particulate organic carbon from macroalgae (`POCMAC`) is computed by modifying the existing routine in `trcsink.F90`. Unlike standard detritus, macroalgal detritus is split into multiple size classes with distinct sinking speeds.

This approach captures the heterogeneity in particle size, distributing the total detrital flux across multiple sinking speed. The sum of all `distribws` weights equals 1.

---

## Implementation

The number of iterations for vertical sinking is adapted per grid point to satisfy a CFL-like condition:

```
iiter(i,j) = max(iiter(i,j), floor(w(i,j,k) / (0.5 * e3t(i,j,k) * rday / rsfact)) + 1)
```

For macroalgal particles we added `nitermaxmac` that allow bigger time splitting than std PISCES:

```fortran
IF (omtype .EQ. 2) THEN
   iiter(:,:) = MIN( iiter(:,:), nitermaxmac )
ENDIF
```


Several calls to `trc_sink` are used, each for a different particle class:

```fortran
CALL trc_sink( kt, Kbb, Kmm, wsmacs1, zsinkingmacs1 , jpdetmac, rfact2, 2 , 0.1*r_exptodeep) ! small POCMAC, class 1
CALL trc_sink( kt, Kbb, Kmm, wsmacs2, zsinkingmacs2 , jpdetmac, rfact2, 2 , 0.1*r_exptodeep)
CALL trc_sink( kt, Kbb, Kmm, wsmacb1, zsinkingmacb1 , jpdetmac, rfact2, 2 , 0.3*r_exptodeep) ! large POCMAC, class 1
CALL trc_sink( kt, Kbb, Kmm, wsmacb2, zsinkingmacb2 , jpdetmac, rfact2, 2 , 0.5*r_exptodeep) 
```

A large part of POC settles directly on the floor next to seaweed, so only `r_exptodeep` (15%) is considered to be able to sink to the bottom of the grid cell (Filbee-Dexter et al. 2024).
Each call distributes a portion of the tracer mass based on `distribws`. The total detritus is conserved: each portion sinks independently but all come from the same `tr(jpdetmac)` tracer.

---

## Variable Mapping

| Variable              | Description                                                                 
|-----------------------|-----------------------------------------------------------------------------
| `jpdetmac`            | Index of macroalgal detritus tracer                                         
| `pwsink(ji,jj,jk)`    | Sinking speed for a given size class and location                           
| `distribws`           | Fraction of tracer assigned to this sinking class                           
| `zsinkingmac*`        | Diagnostic array for storing vertical flux of each size class                                             
| `omtype`              | Type of sinking: `1=standard`, `2=macroalgae`                               
| `iiter(ji,jj)`        | Number of sinking sub-steps to satisfy CFL-like condition                   
| `nitermaxmac`         | Maximum number of iterations for macroalgae particle sinking                



---

## Input Parameters (from `nampisdetmac` namelist)

| **Variable in Fortran**     | **Description**                     | **Values**             |
|-----------------------------|-------------------------------------|---------------------------------|
| `ssmacs1 = wsmacs1`      | Sinking speed small particle class1                 | -          |
| `ssmacs2 = wsmacs2`      | Sinking speed small particle class2                 | -          |
| `ssmacb1 = wsmacb1`      | Sinking speed big particle class1                   | -          |
| `ssmacb1 = wsmacb1`      | Sinking speed big particle class2                   | -          |

## Input Parameters (from `namtrc_snk` namelist)

| **Variable in Fortran**     | **Description**                     | **Values**             |
|-----------------------------|-------------------------------------|---------------------------------|
| `nitermaxmac`      | Maximum number of iterations for macroalgae particle sinking                   | -          |

`r_exptodeep` is currently hard coded in `p4zsinkmac.F90`.

## Reference
Filbee-Dexter, Karen, Albert Pessarrodona, Morten F. Pedersen, et al. ‘Carbon Export from Seaweed Forests to Deep Ocean Sinks’. Nature Geoscience 17, no. 6 (2024): 552–59. https://doi.org/10.1038/s41561-024-01449-7.
