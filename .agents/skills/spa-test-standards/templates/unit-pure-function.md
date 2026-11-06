# Template: pure function (validation, calculation, formatting)

**When to use:** pure logic extracted into an exported function (form validation, data normalization, calculation, text formatting) — no `mount`, no mock, call the function directly.

```js
import { describe, it, expect } from 'vitest'
import { myPureFunction } from '../myPureFunction'

describe('myPureFunction', () => {
  it('should <behavior in the normal case>', () => {
    expect(myPureFunction(validInput)).toBe(expectedValue)
  })

  it('should <behavior at the boundary/error case>', () => {
    expect(myPureFunction(invalidInput)).toBeNull() // or the real fallback value
  })
})
```

## Mutation-proofing: illustrative examples

### Boundary rule by count (pluralization, quantity thresholds)
When the function varies the text/result based on a count, each range is a different mutation (`=== 1` becomes `<= 1`, `> 1` becomes `>= 1`, etc.) — test each range in isolation, not just "the generic case":

```js
describe('formatCount', () => {
  it('should use singular wording for a count of 1', () => {
    expect(formatCount(1, 'item')).toBe('1 item')
  })

  it('should use plural wording for a count of 2', () => {
    expect(formatCount(2, 'item')).toBe('2 items')
  })

  it('should use plural wording for counts above 2', () => {
    expect(formatCount(5, 'item')).toBe('5 items')
  })

  it('should return an empty string when count is 0 and no fallback is provided', () => {
    expect(formatCount(0, 'item')).toBe('')
  })

  it('should not throw when called without arguments', () => {
    expect(formatCount()).toBe('')
  })
})
```

### Zeroing out at a boundary value + exclusion by flag
When the function zeroes out/discards a value outside a range, or excludes a field because of a flag, test both sides of each comparison and the effect of each flag in isolation — testing only "everything within range" or "all flags on" lets mutations in comparisons (`< 0` → `<= 0`, `> 100` → `>= 100`) and conditional spreads slip through:

```js
describe('calculateAdjustedValue', () => {
  it('should zero out a negative value', () => {
    expect(calculateAdjustedValue({ value: -10 })).toBe(0)
  })

  it('should zero out a value above the allowed maximum', () => {
    expect(calculateAdjustedValue({ value: 150, max: 100 })).toBe(0)
  })

  it('should keep a value within bounds unchanged', () => {
    expect(calculateAdjustedValue({ value: 42, max: 100 })).toBe(42)
  })
})

describe('calculateRowTotal', () => {
  it('should ignore the "optionalField" field when its flag is off, even if present in the input', () => {
    const row = { optionalField: 999, hasOptionalField: false, otherField: 10 }

    expect(calculateRowTotal(row)).toEqual({ total: 10 })
  })

  it('should include the "optionalField" field when its flag is on', () => {
    const row = { optionalField: 999, hasOptionalField: true, otherField: 10 }

    expect(calculateRowTotal(row)).toEqual({ total: 1009 })
  })
})
```

**General principle:** cover `0`, negative, above the maximum, empty string, empty array, `undefined`/`null`, one item vs. multiple items — each boundary catches a different comparison mutation.
