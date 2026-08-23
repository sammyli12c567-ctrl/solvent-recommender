# Solvent & Reaction Mechanism Recommender

A Python/RDKit tool that predicts whether an SN2 or E2 mechanism is favored for
a given primary alkyl halide substrate and reagent, and recommends a solvent
based on that prediction.

This started as a simple rule-based script (if nucleophile is strong and
carbon is primary/secondary, predict SN2) and has been rebuilt into a
multi-factor scoring system, after working through the underlying organic
chemistry in more depth.

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
   bonus toward E2, reflecting that elimination is entropically favored.
5. **Convert scores to a percentage split** between SN2 and E2, rather than
   returning one fixed answer — real reactions typically produce a mixture
   of products, so a percentage is a more honest representation than a
   single mechanism.
6. **Check leaving group quality** (I > Br > Cl >> F) as a separate note,
   since a poor leaving group slows down all mechanisms roughly equally
   rather than favoring one over another.
7. **Recommend a solvent** based on the predicted mechanism (protic solvents
   favor E2, aprotic solvents favor SN2), ranked by safety/toxicity.

**Note on scope**: this version only handles primary substrates (SN2/E2
prediction). Secondary and tertiary substrates (SN1/E1) are in progress.

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

## What's next

- Secondary and tertiary substrate handling (SN1/E1 prediction)
- Testing against known textbook reaction examples
- Possible PubChem integration for live solvent property lookup
