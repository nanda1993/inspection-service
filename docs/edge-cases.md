# Edge cases for `classifyDefect(reading, partType)`

Written from the signature and `docs/tolerance-spec.md` (v1.2), before any test. `Expected` is what the specification says, not what
the code does today. Limits: shaft 24.95 to 25.05, bore 39.98 to 40.02, bracket 12.60 to 12.80, spacer 6.28 to 6.42, sleeve 19.02 to
19.08, shim 1.55 to 1.65, washer 3.05 to 3.35. Compare at 0.001 mm (4.4). The two sensor-fault checks (4.6) use the reading as sent.

| Case | Input | Expected | Why it matters | Family |
|---|---|---|---|---|
| exactly on the upper limit | 25.05, shaft | rework | 4.2 and 4.4: a part on a limit is outside, so a too-big shaft must not be accepted | boundary |
| a step inside the upper limit | 25.049, shaft | pass | 4.1: the last passing reading, so a limit that moved inwards would scrap good parts | boundary |
| a step above the upper limit | 25.051, shaft | rework | 4.2: just outside is too big and can be machined down | boundary |
| exactly on the lower limit | 24.95, shaft | scrap | 4.3 and 4.4: a part on the lower limit is outside, and a wrong pass ships a too-small part | boundary |
| a step inside the lower limit | 24.951, shaft | pass | 4.1: the first passing reading above the lower limit | boundary |
| a step below the lower limit | 24.949, shaft | scrap | 4.3: just outside is too small and cannot be fixed | boundary |
| exactly the nominal size | 25, shaft | pass | 4.1: the plain, expected case, and the anchor for the other rows | boundary |
| awkward size: sleeve exactly on the upper limit | 19.08, sleeve | rework | 4.4 holds for every part: 19.05 + 0.03 is 19.080000000000002 in floating point, so a raw comparison lets 19.08 through | awkward size |
| awkward size: sleeve a step inside the upper limit | 19.079, sleeve | pass | 4.1: the awkward limit must not reject a good part | awkward size |
| awkward size: spacer exactly on the lower limit | 6.28, spacer | scrap | 4.3 and 4.4: 6.35 - 0.07 is 6.279999999999999 in floating point, so a raw comparison accepts 6.28 | awkward size |
| awkward size: bracket exactly on the upper limit | 12.8, bracket | rework | 4.4: 12.7 + 0.1 is 12.799999999999999, so this one is only right by luck with a raw comparison | awkward size |
| awkward size: shim exactly on the upper limit | 1.65, shim | rework | 4.4: 1.6 + 0.05 is 1.6500000000000001 in floating point | awkward size |
| awkward size: washer exactly on the lower limit | 3.05, washer | scrap | 4.3 and 4.4: 3.2 - 0.15 is 3.0500000000000003 in floating point | awkward size |
| a fourth decimal that rounds onto the upper limit | 25.0496, shaft | rework | 4.4: the gauge works to 0.001, so 25.0496 rounds to 25.050 and sits on the limit | boundary |
| a fourth decimal that rounds inside the upper limit | 25.0494, shaft | pass | 4.4: rounds to 25.049, which is inside, so rounding must go to the nearest 0.001 and not up | boundary |
| a fourth decimal that rounds onto the lower limit | 24.9504, shaft | scrap | 4.4: rounds to 24.950, which sits on the lower limit | boundary |
| the gauge sent null | null, shaft | recheck | 4.5: nobody should scrap or pass a part nobody measured | no reading |
| the gauge sent undefined | undefined, shaft | recheck | 4.5: a missing value is not a verdict | no reading |
| the gauge sent NaN | NaN, shaft | recheck | 4.5: NaN fails every comparison, so without a check it falls through to a verdict | no reading |
| the gauge sent text, not a number | '25.04', shaft | recheck | 4.5: anything that is not a number, even text that looks like one | no reading |
| a reading of zero | 0, shaft | recheck | 4.6: a sensor fault, not a too-small part, so it must not be scrapped | impossible reading |
| a negative reading | -0.5, shaft | recheck | 4.6: a size below zero cannot exist, and the parser lets it through on purpose (2.3) | impossible reading |
| negative infinity | -Infinity, shaft | recheck | 4.6: zero or less, so a sensor fault | impossible reading |
| a reading far too big | 9999, shaft | recheck | 4.6: more than twice the nominal size, so a sensor fault and not a rework | impossible reading |
| infinity | Infinity, shaft | recheck | 4.6: infinite is a sensor fault, not a too-big part | impossible reading |
| exactly twice the nominal size | 50, shaft | rework | 4.6: exactly twice is possible and is classified under 4.2, so the check must be "more than" | impossible reading |
| a step over twice the nominal size | 50.001, shaft | recheck | 4.6: the first impossible reading above the cut-off | impossible reading |
| over twice the nominal, but rounds down to it | 50.0004, shaft | recheck | 4.6: the check looks at the reading as sent, before rounding, so this is a fault even though it rounds to 50.000 | impossible reading |
| a tiny positive reading that rounds to zero | 0.0004, shaft | scrap | 4.6: it is above zero as sent, so it is not a fault, and after rounding it is far below the lower limit | impossible reading |
| exactly twice the nominal, awkward size | 25.4, bracket | rework | 4.6: twice 12.7 is 25.4, the cut-off for a part whose size is not a round number | impossible reading |
| a part type that is not in the table | 25, gear | throws | 4.7: a data error, reported with an UnknownPartError | unknown part |
| a part type in the wrong case | 25, Shaft | throws | 4.7: the table keys are exact, and 'Shaft' is not 'shaft' | unknown part |
| unknown part and no reading | null, gear | throws | 4.7: the part is checked before the reading, so the answer is the error and not recheck | unknown part |
| unknown part and an impossible reading | 0, gear | throws | 4.7: the part is checked first, so the error beats the sensor fault | unknown part |
| no part type at all | 25, undefined | throws | 4.7: undefined is not in the table | unknown part |
| a name the object inherits, not one in the table | 25, constructor | throws | 4.7: `constructor`, `toString` and `__proto__` exist on every object but are not parts, so a plain property lookup can mistake them for one | unknown part |
