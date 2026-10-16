# Park Accessibility and Gender Differences in Seoul

Do men and women of different ages respond differently to park features when deciding which park to visit and how far to travel? This project models the travel cost visitors pay to reach **274 urban parks in Seoul** from all **25 districts (gu)**. It tests whether the effects of park amenities (green space, facilities, transit access, size) differ by sex and age.

## Data

- **Visitor flows (2019):** visitor counts for each origin district × park pair, split into 28 sex × age groups (men and women, 14 age bands from 0–9 to 70+). That gives 6,850 origin–park pairs and **191,750 non-zero observations**.
- **Travel cost:** `c_eq_min`, the generalized travel cost in equivalent minutes from the origin district to the park.
- **Park attributes:** built from park inventory variables, matching Table 1 of the paper:

| Variable | Definition |
|---|---|
| `green_ratio` | Green area / park area (%) |
| `user_ratio` | Open space + paths + walking trails + shade canopy / park area (%) |
| `facility_number` | Gym/track, pools, sports facilities, playgrounds, cafeterias, kiosks |
| `parking_number` | Parking capacity |
| `transit_number` | Bus stops + subway stations within 500 m |
| `park_area_ha` | Park area (ha) |

The data are not included in this repository.

## Method

**Cross-classified random-effects model (CCREM).** Each observation is an origin district × park × sex-age group cell. Cells are nested in two non-hierarchical groupings, parks and origin districts, so the model uses crossed random intercepts for both:

```
c_eq_min ~ female + age + age² + female×age + female×age²
         + Σ amenity × {1, female, age, age², female×age, female×age²}
         + (1 | park) + (1 | origin district)
```

- 41 fixed-effect terms. The model is weighted by visitor count (rescaled to mean 1) and fit by REML with `lme4`.
- **Age coding fix:** the open-ended 70+ band is coded as age 75. An earlier version misread the column code and set it to 35.
- **Marginal effects** of each amenity by sex and age group, with **delta-method** 95% confidence intervals from the fixed-effect covariance matrix.
- **Accessibility measures:** potential accessibility is a logsum over all parks for each origin (θ = 0.10, with 0.05 and 0.20 as sensitivity values). Realized accessibility is the visitor-weighted mean travel cost.

**Diagnostics and robustness**
- Non-singular fit. BOBYQA and Nelder–Mead optimizers agree (maximum fixed-effect difference 4 × 10⁻⁸).
- Variance inflation factors for the full interaction design.
- A likelihood-ratio test of the female × age terms.
- An alternative **origin fixed-effects** model (24 district dummies) with **park-clustered standard errors** (CR1), computed by hand.

## Main results

| | |
|---|---|
| Observations / parks / origin districts | 191,750 / 274 / 25 |
| ICC: park / origin district | 0.64 / 0.12 |
| Marginal R² / conditional R² | 0.12 / 0.78 |
| LRT for female × age, female × age² | χ²(2) = 2.98, p = 0.23 |

- **Most variation is between parks.** Park-level differences explain about 64% of the residual variance, versus 12% for origin districts.
- **Transit access and park size matter most.** A one-SD increase in transit stops nearby (≈ 24 stops) is associated with **4.7 fewer equivalent minutes** of visitor travel cost (p = 0.001). A one-SD increase in park area (≈ 22 ha) is associated with **3.5 fewer minutes** (p = 0.015). Green ratio, user space, facilities, and parking have no significant main effects.
- **Gender differences appear through amenities, not directly.** The female × age terms do not improve fit (LRT p = 0.23), but several amenity effects differ by sex:
  - **Transit:** the effect is significantly stronger for women aged 20–59 than for men of the same age (p < 0.05).
  - **Park size:** the effect is weaker for girls under 20 but stronger for women from age 30, and the gap grows with age, largest at 60+ (p < 0.001).
- **Similar potential access, different realized travel.** Potential accessibility is almost the same for men and women (gap below 0.15 equivalent minutes). Women aged 30–69 travel about **1.3–1.5 minutes less** than men to the parks they actually visit.
- The origin fixed-effects model with park-clustered errors gives consistent estimates (R² = 0.27).

## Repository contents

| File | Description |
|---|---|
| `ccrem_park_accessibility.Rmd` | Full pipeline: data reshaping, interaction terms, VIF, CCREM fit and diagnostics, LRT, marginal effects with delta-method CIs, LaTeX table rows, figures, accessibility measures, and the fixed-effects robustness model |
| `ccrem_park_accessibility.html` | Knitted report with all output. Download it and open it in a browser; GitHub shows HTML as source. |

## How to run

1. Put `FINAL_dataset_2019only.csv` (the origin–park file in wide format) in `data/`.
2. Open `park_accessibility.Rproj` in RStudio. File paths are set in the `params` block of the Rmd and are relative to the project folder.
3. Install the packages: `install.packages(c("lme4", "lmerTest", "ggplot2"))`. All other steps use base R.
4. Knit `ccrem_park_accessibility.Rmd`. Tables (`.csv`, `.tex`) and figures (`.png`) are written to `results/`. The CCREM fit takes several minutes.

## Author

Dawon Oh: data processing, model specification, estimation, and visualization.
