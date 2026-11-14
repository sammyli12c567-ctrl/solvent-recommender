# Solvent & Reaction Mechanism Recommender

A Python/RDKit tool for organic chemistry students. Give it an alkyl halide
(as a SMILES string), a nucleophile or base, and a temperature. It scores the
competing substitution and elimination mechanisms, reports them as a
percentage split where it can, and recommends solvents ranked by safety and
toxicity (CHEM21).

This started as a simple rule-based script (if nucleophile is strong and
carbon is primary/secondary, predict SN2) and has been rebuilt into a
multi-factor scoring system, after working through the underlying organic
chemistry in more depth.

## Try it

The notebook `T_project47.ipynb` is the current version. It was written to run
in Google Colab: open it there and run the cells in order (it installs RDKit
and writes its solvent table to `/content/sample_data/`). When asked, enter:

- the molecule as SMILES, for example `CCCBr`
- a nucleophile, for example `CN` (it knows: oh, cn, i, br, cl, sh, n3, aco,
  otbu, oet, nh2, h2o, etoh, f, scn)
- the temperature in C, for example `25`

Example (`CCCBr`, `CN`, 25 C): SN2 score 4 and E2 score 2, which becomes
`{'SN2': 66.7, 'E2': 33.3}`, plus a leaving-group note (good), a short
explanation of why, and aprotic solvents listed from least to most hazardous.

`reaction_engine_v2 (1).ipynb` is the earlier version (Aug 23), kept for
history.

## How it works

1. **Parse the molecule** (SMILES input) with RDKit, identify the leaving
   group and the reactive carbon.
2. **Classify the substrate** as primary, secondary, or tertiary based on
   the number of carbon substituents on the reactive carbon.
3. **Score the reagent** against several independent factors:
   - Nucleophilicity (qualitative: strong / moderate / weak)
   - Basicity (qualitative: strong / moderate / weak) — tracked *separately*
     from nucleophilicity, since these are not the same property (e.g.
     cyanide is a strong nucleophile but a comparatively weak base)
   - Steric bulk of the reagent (a bulky reagent can't easily reach the
     carbon for backside attack, favoring E2 instead)
4. **Factor in temperature** — elevated temperature (>50C) gives a small
   bonus to the elimination pathways, reflecting that elimination is
   entropically favored.
5. **Convert scores to a percentage split** between the competing mechanisms
   (SN2/E2 for primary substrates, SN1/E1/E2 for tertiary), rather than
   returning one fixed answer — real reactions typically produce a mixture
   of products, so a percentage is a more honest representation than a
   single mechanism. Secondary substrates show raw scores for all four
   mechanisms.
6. **Check leaving group quality** (I > Br > Cl >> F) as a separate note,
   since a poor leaving group slows down all mechanisms roughly equally
   rather than favoring one over another.
7. **Recommend a solvent** based on the predicted mechanism (aprotic
   solvents favor SN2/E2, protic solvents favor SN1/E1), ranked by
   safety/toxicity.

**Note on scope**: handles alkyl halides with F, Cl, Br or I as the leaving
group. Primary substrates are scored for SN2/E2, tertiary for SN1/E1/E2, and
secondary for all four.

## Data sources

- **Solvent safety/toxicity classification**: [CHEM21 Solvent Selection
  Guide](https://doi.org/10.1039/C5GC01008J) (Prat et al., *Green
  Chemistry*, 2016) — a peer-reviewed methodology used in pharmaceutical
  solvent selection, scoring solvents on Safety, Health, and Environment
  criteria. Cost is intentionally not included as a ranking factor, since
  CHEM21 does not include it either.
- **Protic/aprotic classification**: determined by presence of O-H/N-H
  bonds — standard organic chemistry, not from CHEM21.
- **Nucleophilicity/basicity ratings**: qualitative tiers based on general
  chemical knowledge, not derived from a single numeric source. (A more
  rigorous version could use Swain-Scott nucleophilicity constants —
  considered, but left out of this version to keep scope manageable given
  the project timeline.)

## Known limitations

- Point values assigned to basicity, steric bulk, and temperature are
  reasoned judgment calls, not derived from a quantitative kinetic model.
  Directions (e.g. "bulky reagent favors E2") are chemically established;
  the exact magnitude of each factor's contribution is my own estimate.
- The temperature threshold (50C) is a simplification of what is, in
  reality, a continuous relationship (via the Arrhenius equation), not a
  hard cutoff.
- Solvent is intentionally *not* used as an input to the mechanism
  prediction itself — the prediction is based on substrate and reagent
  properties only, and solvent is recommended afterward based on the
  predicted outcome. This is a deliberate design choice (so the tool can
  recommend a solvent rather than requiring the user to already know one),
  but it means the prediction doesn't account for a solvent the user may
  already be planning to use.
- This is a personal learning project, not a substitute for professional
  solvent selection tools. Comparable, more comprehensive tools already
  exist at an industry level — e.g. the [ACS GCI Pharmaceutical Roundtable's
  Solvent Selection Tool](https://learning.acsgcipr.org/), built on the
  CHEM21 dataset by pharmaceutical industry teams. This project was built
  independently, from first principles, as a way to understand the
  underlying chemistry deeply enough to reason about it in code, not to
  compete with those tools in scope or rigor.

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
