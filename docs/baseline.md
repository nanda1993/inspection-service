# Coverage baseline

The team's tests only (`tests/existing`), on the code as shipped. Measured with `npm run coverage -- --team`.

## Coverage numbers

| File | Lines % | Branches % |
|---|---|---|
| classifyDefect.js | 70.4 | 40.0 |
| parser.js | 100.0 | 66.7 |

## Untested gaps

Highest risk first.

| Untested | Lines | If this is wrong in production | Risk |
|---|---|---|---|
| The rework and scrap verdicts: a reading above the upper limit returns `'rework'`, anything else left over returns `'scrap'` | 23-26 | A part goes to the wrong bin: a recoverable oversize part is scrapped and destroyed, or an undersize part that cannot be fixed is sent to rework. Nobody would know, because no test has ever asked for either answer. | high |
| No reading: `null` returns `'recheck'` | 16-17 | A part nobody measured gets a verdict (today a missing reading would fall through to `'scrap'`), so a good part is destroyed because the gauge sent nothing. | high |
| Unknown part type: throws `UnknownPartError` | 13-14 | A wrong part code is not reported as a data error, so the problem is hidden or surfaces as a confusing crash instead of a clear message. | medium |
| Screen text: `labelFor` for unknown outcomes (12-13) and `summarise` (18-25) | 12-13 18-25 | The operator sees a wrong or missing label on the screen; the verdict itself is unaffected. | low |

Coverage cannot see: a branch that was never written, such as the missing handling of `undefined`, `NaN`, zero, negative and oversize readings (spec 4.5 and 4.6) in `classifyDefect`, which gives no line to mark as untested.
