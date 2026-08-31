# GaitSense Reference Gait-Parameter Report

This report compares reference gait parameters with pose-derived estimates. It is a technical PoC comparison, not a clinical validation or medical assessment.

- Rows: 1
- Participants/trial IDs: 1
- Reference source: `gait_parameters.csv`
- Estimate source: `gait_parameters_estimation.csv`

## Walking speed comparison

| ID | Pace | Reference (m/s) | Estimated (m/s) | Absolute error (m/s) | Relative error |
|---|---|---:|---:|---:|---:|
| PA000 | UGS | 2.040 | 1.865 | 0.175 | 8.58% |
| PA000 | FGS | 2.040 | 2.104 | 0.064 | 3.14% |

## Parameter comparison

| ID | Pace | Step ref/est (cm) | Stride ref/est (cm) | Cadence ref/est (steps/min) |
|---|---|---:|---:|---:|
| PA000 | UGS | 54.300 / 76.242 | 110.500 / 161.702 | 133.910 / 133.237 |
| PA000 | FGS | 54.300 / 79.537 | 110.500 / 170.101 | 133.910 / 141.670 |

## Limitations

The checked-in sample contains one row only, so it cannot support regression, generalization, or statistical validation. The reference and estimated values may not represent identical measurement conventions. Use the full dataset and keep participant IDs grouped when evaluating models.
