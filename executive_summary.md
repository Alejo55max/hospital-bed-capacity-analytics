# Executive Summary — Hospital Bed Capacity Analytics

*Condensed from the full capstone report. Full model code: [`hospital_capacity_analysis.ipynb`](hospital_capacity_analysis.ipynb).*

## Industry Problem

The Australian health sector has been under sustained operational and financial pressure for over a decade. In 2023–24, public and private hospitals recorded 12.6 million hospitalisations and delivered 33.9 million patient-days of care across 672 public hospitals alone (AIHW 2024a). Continued growth in the elderly population is intensifying pressure on hospital capacity, workforce and funding (AIHW 2024a, 2019).

Hospitals classify patients into clinically meaningful groups — Australian Refined Diagnosis Related Groups (AR-DRGs) — for planning, reporting and funding (IHACPA 2022). A persistent business challenge is **bed congestion**: high daily patient demand combined with prolonged stays for certain conditions, which reduces throughput and contributes to treatment delays, procedure cancellations and rising operating costs.

Reported hospital bed-day costs vary by jurisdiction and care intensity: NSW Health (2024) reports ~$1,075 AUD/day for an average public hospital bed; the WA Audit Commission (2022) reports ~$2,370 AUD/day; and ICU-level care has been estimated at $4,375–$4,875 AUD/day (Hicks et al. 2019). Long-stay admissions are flagged as one of the most common drivers of disproportionate bed use in Australian public hospitals (WA Audit Commission 2022).

**Business question:** How can hospitals improve bed capacity and reduce congestion by identifying which AR-DRGs contribute most to total patient-day demand?

## Data & Methodology

- **Source:** AIHW Australian Refined Diagnosis Related Groups (AR-DRG) Data Cube, 2023–24 — national admitted-patient activity by DRG, age group, sex and same-day flag.
- **Cleaning:** removed an embedded repeated header row, converted separations/patient-days to numeric, excluded invalid records. Raw dataset: 44,513 rows.
- **Aggregation:** grouped to DRG × Age Group × DRG Partition (dropping groups with fewer than 10 separations) to produce stable, planning-relevant estimates rather than noisy patient-level variation — 11,902 modelling segments.
- **Model:** multiple linear regression predicting **Average Length of Stay (ALOS)** from DRG, age group and DRG partition (one-hot encoded). Linear regression was chosen for transparency and interpretability, which matters for a model meant to support operational decisions, not just to maximise accuracy.

## Model Results

| Metric | Value |
|---|---|
| R² | 0.855 |
| MAE | 1.17 days |
| RMSE | 2.61 days |
| Naive baseline MAE | 4.58 days |
| Error reduction vs. baseline | ~74% |

The model explains roughly 85.5% of the variation in ALOS from case type, age and partition alone, with an average error of about one day per segment — accurate enough to support bed planning, discharge forecasting and congestion monitoring.

## Scenario Analysis

Predicted ALOS was converted into predicted bed-days (patient-days), and a **targeted intervention scenario** was modelled: a 5% and 10% reduction in ALOS across the highest-burden DRG × age segments (rather than a system-wide change). **Haemodialysis (L61Z)** and **pharmacotherapy for neoplastic disorders (R63Z)** consistently emerge as the two largest contributors to predicted bed demand, concentrated in older age groups.

For haemodialysis alone, a 5–10% ALOS reduction was estimated to free approximately **120,000–230,000 bed-days**, against a baseline cost burden of roughly AU$5.9 billion — implying potential savings in the order of AU$300–600 million from this single DRG. Extending the same 5–10% reduction across the top 50 highest-burden DRG × age segments nationally, using a **central planning assumption of $2,500/bed-day** (in line with WA Audit Commission benchmarks), scales to an estimated **287,000–574,000 bed-days freed** and **AU$0.7–1.4 billion** in avoided costs per year.

**These are scenario estimates, not audited financial forecasts.** The bed-days-saved figures are the more robust output (they don't depend on a cost assumption); the $ figures should be read as an order-of-magnitude business case using the cited government benchmark range.

## Recommendations

1. **Prioritise haemodialysis and neoplastic pharmacotherapy DRGs first** — a small number of DRGs account for a disproportionate share of national bed-days, so targeted interventions outperform broad, system-wide programs.
2. **Strengthen discharge planning and outpatient follow-up** for these high-burden segments, with closer coordination between hospital and community healthcare providers.
3. **Give bed management teams the ALOS model as a forecasting tool**, so expected occupancy by DRG and age group can be planned for in advance rather than reacted to.
4. **Report cost impact as a range tied to cited benchmarks**, not a single number, so decision-makers can see the sensitivity of the business case to the cost assumption used.

## Limitations

- The AR-DRG cube is aggregated at the national level — it does not capture hospital-specific factors such as severity, facility type or local bed availability, so the model may underperform for rare, very-long-stay cases (visible as higher prediction variance at extreme ALOS values).
- The $2,500/bed-day assumption is an estimate drawn from three government/clinical sources and will vary by hospital, state and patient complexity.
- Single year of data (2023–24) — the model has not been validated against prior years and represents a snapshot rather than a trend.
- The scenario holds admission volumes constant and does not model second-order effects, such as freed beds being immediately backfilled by demand growth.

## References

- Australian Institute of Health and Welfare (AIHW) 2025, *AR-DRG Data Cubes Version 11.0 (2023–24)*, AIHW, Canberra. https://www.aihw.gov.au/hospitals-data/ar-drg-data-cubes
- AIHW 2025, *Hospitals at a glance 2023–24*, AIHW, Canberra. https://www.aihw.gov.au/hospitals/overview/hospitals-at-a-glance
- AIHW 2024a, *Admitted patient care 2023–24: Australian hospital statistics*, AIHW, Canberra. https://www.aihw.gov.au/hospitals/topics/admitted-patient-care
- Independent Health and Aged Care Pricing Authority (IHACPA) 2022, *Australian Refined Diagnosis Related Groups Version 11.0: Overview and development*. https://www.ihacpa.gov.au/sites/default/files/2022-10/ar-drg_version_110_fact_sheet.pdf
- NSW Health 2024, *Health insurers rorting public hospital beds*, NSW Government media release. https://www.nsw.gov.au/media-releases/health-insurers-rorting-public-hospital-beds
- Western Australia Audit Commission 2022, *Management of Long Stay Patients in Public Hospitals*, Perth. https://audit.wa.gov.au/reports-and-publications/reports/management-of-long-stay-patients-in-public-hospitals
- Hicks, P., et al. 2019, 'The financial cost of intensive care in Australia: a multicentre registry study', *Medical Journal of Australia*, vol. 211.
- James, G., Witten, D., Hastie, T. & Tibshirani, R. 2013, *An Introduction to Statistical Learning*, Springer.
- Pedregosa, F., et al. 2011, 'Scikit-learn: Machine Learning in Python', *Journal of Machine Learning Research*, vol. 12.

---
*Capstone project — Master of Business Analytics, Kaplan Business School (2025–26).*
