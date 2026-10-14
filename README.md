## Consistency check against published decision rules

This tool's scoring was compared with decision rules from Master Organic
Chemistry (James Ashenhurst): "Wrapup: The Key Factors For Determining
SN1/SN2/E1/E2" and "Deciding SN1/SN2/E1/E2 (1): The Substrate". This checks
that the code follows those rules. It is **not** validation against
experimental data.

Following advice from Prof. Zheng (WashU) to test against established
examples, I ran 9 cases:

| Substrate | Conditions | Expected (per source) | Scores | Status |
|---|---|---|---|---|
| Primary | CN-, not bulky, rt | SN2 dominant | SN2=4, E2=2 | Consistent |
| Primary | OtBu-, rt | E2 favored | E2=6, SN2=2 | Consistent |
| Primary | Br-, no heat | Substitution | SN2=3, E2=2 | Consistent |
| Tertiary | EtOH, rt | SN1 favored | SN1=3, E1=2 | Consistent |
| Tertiary | OtBu-, heated | E2 favored | E2=8, E1=7, SN1=1 | Partly: E1 too close |
| Secondary | OtBu- | E2 tips ahead | E2=3, E1=2, SN1=1 | Consistent |
| Secondary | OEt- | Close SN2/E2 split | E2=2, SN2=1 | Consistent |
| Secondary | OH- | SN2/E2 mixture | all four = 2 | Needs review: SN1/E1 should be lower |
| Secondary | CN- | SN2 over E2 | SN1=SN2=2, E1=E2=1 | Needs review: SN1 shouldn't compete |

### A gap this process caught
Tertiary scoring originally ignored E2, based on a wrong assumption that
tertiary substrates can't undergo E2. Testing against the source's rule
("tertiary + strongly basic nucleophile -> expect E2") exposed it, and E2
scoring was added to the tertiary branch.

### Limitations
- One source, 9 cases, none held out.
- Scores come from hand-assigned points, not calibrated probabilities.
- No experimental data yet.

### Next steps
Collect 50-100 ground-truth reactions, replace hand-assigned points with a
probabilistic model, and report calibration.
