## Verification against published decision rules

Following advice from Prof. Zheng (WashU) to test predictions against
established examples, this tool's scoring logic was checked against
decision rules published in Master Organic Chemistry (James Ashenhurst),
specifically:
- "Wrapup: The Key Factors For Determining SN1/SN2/E1/E2"
- "Deciding SN1/SN2/E1/E2 (1): The Substrate"

### Primary substrates
| Test case | Expected (per source) | Result |
|---|---|---|
| Strong nucleophile (CN-), not bulky, room temp | SN2 dominant | SN2=4, E2=2 -- matches |
| Strong, bulky base (OtBu-), room temp | E2 favored despite primary substrate | E2=6, SN2=2 -- matches |
| Moderate reagent (Br-), no elevated temp | Substitution favored by default | SN2=3, E2=2 -- matches |

### Tertiary substrates
| Test case | Expected (per source) | Result |
|---|---|---|
| Weak nucleophile/weak base (EtOH), room temp | SN1 favored | SN1=3, E1=2 -- matches |
| Strong, bulky base (OtBu-), heated | E2 favored (heat reinforces E1 over SN1, but does not override E2) | E2=8, E1=7, SN1=1 -- matches, after correcting an earlier gap where E2 was not scored at all for tertiary substrates |

### Secondary substrates
| Test case | Expected (per source) | Result |
|---|---|---|
| Bulky alkoxide (OtBu-) | Borderline SN2/E2, bulk tips toward E2 | E2=3, E1=2, SN1=1, SN2=0 |
| Non-bulky alkoxide (OEt-) | Borderline SN2/E2, closer split | E2=2, SN2=1 |
| Strong nucleophile + strong base (OH-) | Genuine mixture, no dominant mechanism | SN1=E1=SN2=E2=2 (exact tie) -- matches the source's explicit prediction |
| Strong nucleophile, weak base (CN-) | SN1/SN2 favored over E1/E2 | SN1=SN2=2, E1=E2=1 -- matches |

### Known gap this process caught
Initial tertiary-substrate scoring only considered SN1 and E1, based on an
earlier (incorrect) assumption that tertiary substrates couldn't undergo
E2. Testing against the source's explicit claim ("tertiary + strongly
basic nucleophile -> expect E2") surfaced this gap; E2 scoring was added
to the tertiary branch as a result. This is a good example of why
verification against outside sources matters -- the bug wasn't visible
from the code alone.
