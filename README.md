# Number System Converter & Calculator

A self-contained browser application for converting values between binary, octal, decimal, and hexadecimal, evaluating mixed-base arithmetic expressions, and learning radix-complement subtraction through interactive step-by-step views.

The application is implemented in [app.html](app.html). It requires no server, package installation, database, or internet connection.

## Features

- Convert values between binary, octal, decimal, and hexadecimal in real time.
- Accept integer and fractional values, including negative values, in all four supported bases.
- Preserve exact values with `BigInt` rational arithmetic instead of floating-point arithmetic.
- Display all four converted representations for every input.
- Copy generated results by clicking result tiles; click again to collapse expanded values.
- Build expressions with input-variable, operator, parenthesis, Backspace, and Clear buttons.
- Evaluate addition, subtraction, multiplication, division, unary signs, and parentheses with standard precedence.
- Support mixed-base expressions while preserving each input's original base.
- Show positional conversion breakdowns and final arithmetic calculations.
- Calculate radix-specific complements for every active non-negative fixed-point input:
  - Binary: 1's and 2's complements
  - Octal: 7's and 8's complements
  - Decimal: 9's and 10's complements
  - Hexadecimal: 15's and 16's complements
- Provide radix tabs for viewing complement cards in one selected base at a time.
- Provide Minuend `X` and Subtrahend `Y` selectors for complement subtraction.
- Show diminished-radix and radix subtraction methods, including carry handling, negative re-complementing, and radix-point alignment.
- Add and remove input rows while keeping at least three rows available.
- Clear all input values with one control.
- Switch between light and dark themes.
- Adapt the interface for desktop and mobile screens.

## System Requirements

Any computer, tablet, or phone with a current browser supporting JavaScript ES2020, `BigInt`, DOM events, and the Clipboard API. Current versions of Chrome, Edge, Firefox, and Safari are suitable.

Open `app.html` directly in a browser. No build command or development server is required.

## Supported Number Systems

| Number system | Radix | Accepted digits | Complement pair |
|---|---:|---|---|
| Binary | 2 | `0-1` | 1's / 2's |
| Octal | 8 | `0-7` | 7's / 8's |
| Decimal | 10 | `0-9` | 9's / 10's |
| Hexadecimal | 16 | `0-9`, `A-F`, or `a-f` | 15's / 16's |

Converter inputs may contain an optional leading minus sign and one radix point. Examples include `10.5`, `1010.1`, `2A.8`, `10.`, and `.5`. Leading and trailing spaces are removed before validation.

Complement calculations accept non-negative integer and fractional values. They use the first five fractional digits; repeating values are truncated at the fifth digit instead of being rejected. This five-digit rule applies only to the complement module; the converter and expression calculator retain their exact rational values. Negative values remain supported by the converter and expression calculator but are rejected by the complement module because complement subtraction is defined here for non-negative fixed-point operands.

## User Interface

The page contains five main areas:

1. **Header**: title, student/course information, and theme toggle.
2. **Input rows**: three rows initially, each with a source-base selector, value field, validation message, and four result tiles.
3. **Expression builder**: read-only expression display and click-built input/operator controls.
4. **Arithmetic result**: expression output, conversion solution, and results in all four bases.
5. **Complements & Subtraction**: radix tabs, per-input complement cards, and selectable X/Y subtraction.

### Input management

- `Add Input` creates another input row.
- `Remove` deletes a row only when more than three rows exist.
- Rows are renumbered after additions and removals.
- `Clear Values` empties all value fields and clears dependent output.
- Changing a base updates that row immediately.

### Result tiles

Each populated input produces Binary, Octal, Decimal, and Hexadecimal tiles. Collapsed tiles show up to five fractional digits. Clicking a tile expands it to the generated value, copies it when permitted, and briefly displays `Copied`.

## Conversion Algorithm

Every value is represented as an exact rational object:

```text
{ numerator: BigInt, denominator: BigInt }
```

### Source-base conversion

```text
Trim the input and separate an optional leading minus sign.
Split the value into whole and fractional digits.
Process joined digits from left to right:
    numerator = numerator * sourceBase + digitValue
Set denominator = sourceBase ^ fractionalDigitCount.
Apply the sign to numerator.
```

### Target-base conversion

```text
Separate the sign and use the absolute numerator.
Convert the integer part using repeated division and remainders.
Convert the fractional remainder using repeated multiplication by targetBase.
Stop after 32 generated fractional digits.
Append ... when a remainder is still present.
Restore the sign.
```

The implementation uses `baseStringToRational()` and `rationalToBaseString()`. `reduceRational()` and `gcd()` keep rational values in lowest terms.

## Expression Calculator

Non-empty input rows become variables in order: `a`, `b`, `c`, and so on. The builder prevents invalid next tokens and recalculates after input or builder changes.

Supported operations are addition, subtraction, multiplication, division, unary plus/minus, and parentheses. Evaluation order is:

1. Parentheses and unary signs
2. Multiplication and division, left to right
3. Addition and subtraction, left to right

`tokenizeExpression()` recognizes expression tokens and `evaluateExpression()` uses recursive descent. `computeRational()` performs exact operations. Division by zero, invalid expressions, unavailable variables, and incomplete expressions produce errors.

The arithmetic result includes:

- The original expression with base annotations
- A positional conversion-to-decimal table
- The final calculation using decimal values
- Binary, Octal, Decimal, and Hexadecimal result tiles

## Complement Viewer

The complement viewer is independent from subtraction. It processes every active input that is a valid non-negative fixed-point value. Each input is displayed in its own card with two columns: diminished-radix complement on the left and radix complement on the right.

Select Binary, Octal, Decimal, or Hexadecimal with the radix tabs. The selected radix is used to render every active input card.

### Diminished-radix complement

The left column shows each digit operation:

```text
(radix - 1) - digit = complementDigit
```

It then displays the complete diminished-radix complement.

### Radix complement

The right column shows:

```text
diminished-radix complement
Add 1
Result
```

Labels change with the selected radix. Decimal displays `9's complement` and `10's complement`; hexadecimal displays `15's complement` and `16's complement`.

Complement work uses at most five fractional digits. Repeating values are truncated at the fifth digit, not rounded. Values with fewer fractional digits retain their available precision.

## Interactive Complement Subtraction

This module is separate from the per-input complement viewer. The controls select:

- **Minuend X**: the input being reduced.
- **Subtrahend Y**: the input being subtracted.

The subtraction recalculates whenever an input, source base, radix tab, or selector changes.

### Fixed-width alignment

For selected values `X` and `Y`, the whole-number width is aligned to the larger whole-number digit count in the selected radix. Fractional width is aligned separately to the larger fractional digit count, up to five places. Leading zeroes and trailing fractional zeroes are added before complement calculation and addition, so values such as `10.5` and `2.25` become `10.50` and `02.25`.

### Diminished-radix method

The first method calculates `X + Y_(r-1)'s`. If an end carry is produced, it is added around to the least significant digit. If no end carry is produced, the result is negative and its magnitude is obtained by re-complementing the sum.

### Radix method

The second method calculates `X + Y_r's`. If an end carry is produced, the carry bit is discarded. If no end carry is produced, the result is negative and its magnitude is obtained by re-complementing the sum.

`complementSubtraction(a, b, base, width, fractionWidth)` returns padded fixed-point operands, digit operations, complements, raw sums, carry flags, and final results. The same logic works regardless of which active input is selected as `X` or `Y`.

## Event Flow

```text
Input or base selector changes:
    Update the row's four conversion results.
    Refresh expression variables and arithmetic output.
    Recalculate every active input's complement cards.
    Refresh X and Y selector options.
    Recalculate the selected subtraction.

Radix tab changes:
    Re-render all active input complement cards in that radix.
    Re-render the selected X - Y subtraction walkthrough in that radix.
    Truncate repeating fractional representations to five digits for complement work.

X or Y selector changes:
    Recalculate the selected subtraction operands.
```

## Program Structure

```text
number-system-converter/
├── app.html
├── README.md
└── sample-outputs/
    ├── Flowchart.png
    ├── decimal-input.png
    ├── fractional-decimal-input.png
    ├── hexadecimal-input.png
    ├── invalid-input.png
    └── long-fractional-output.png
```

All application HTML, CSS, and JavaScript are embedded in `app.html`.

The complete Mermaid-aligned flowchart is available in [flowchart.html](flowchart.html) and is linked from the app header. It includes the three application modules, validation branches, complement carry decision, and user loop.


### Main implementation functions

- `baseStringToRational(rawValue, base)`: parses a source value into an exact rational.
- `rationalToBaseString(rational, base)`: formats a rational in a target base.
- `validateInput(rawValue, base)`: validates digits and syntax.
- `updateRow(card)`: updates one input row.
- `computeRational(a, b, op)`: performs exact rational arithmetic.
- `collectOperands()`: gathers expression operands.
- `evaluateExpression(tokens, operands)`: evaluates the expression tree.
- `fixedPointParts(rational, base)`: converts a value to whole and up-to-five-digit fractional parts for complement work.
- `fixedPointString(value, base, width, fractionWidth)`: formats aligned fixed-point digit strings.
- `complementValueData(value, base, width, fractionWidth)`: generates one input's complement data.
- `complementSubtraction(a, b, base, width, fractionWidth)`: performs fixed-width fixed-point complement subtraction.
- `renderInputComplements(operands, base)`: renders all active input cards.
- `renderSubtraction(operands)`: renders the selected X-minus-Y walkthrough.
- `updateComplements()`: coordinates complement viewer and subtraction updates.

## Validation and Error Handling

- Empty converter inputs are treated as cleared rows.
- Invalid digits show a base-specific message and clear that row's results.
- Converter values may be negative or fractional.
- Complement inputs must be non-negative integers or fractions in the selected radix.
- Complement fractional precision is limited to five digits after the radix point; repeating values are truncated rather than rejected.
- Invalid complement input clears both complement output areas until valid input is available.
- The arithmetic calculator reports malformed expressions, missing variables, incomplete expressions, and division by zero.
- At least two non-empty inputs are needed for an expression or complement subtraction.
- The expression builder supports up to 26 non-empty variables, from `a` through `z`.
- Prefixes such as `0b`, `0o`, and `0x` are not accepted.

## Test Cases

### Conversion tests

| Test | Input | Source base | Binary | Octal | Decimal | Hexadecimal |
|---:|---|---:|---|---|---:|---|
| 1 | `42` | 10 | `101010` | `52` | `42` | `2A` |
| 2 | `101010` | 2 | `101010` | `52` | `42` | `2A` |
| 3 | `52` | 8 | `101010` | `52` | `42` | `2A` |
| 4 | `2A` | 16 | `101010` | `52` | `42` | `2A` |
| 5 | `-15` | 10 | `-1111` | `-17` | `-15` | `-F` |
| 6 | `0` | 10 | `0` | `0` | `0` | `0` |
| 7 | `10.5` | 10 | `1010.1` | `12.4` | `10.5` | `A.8` |
| 8 | `-2A.8` | 16 | `-101010.1` | `-52.4` | `-42.5` | `-2A.8` |

Invalid examples include `10201` in Binary, `1G` in Hexadecimal, and `811.478` in Octal.

### Arithmetic tests

| Test | Inputs | Expression | Decimal result | Binary | Octal | Hexadecimal |
|---:|---|---|---:|---|---|---|
| A1 | `10`@10, `20`@10, `30`@10 | `a + b + c` | `60` | `111100` | `74` | `3C` |
| A2 | `FF`@16, `1`@10, `10`@8 | `a - b - c` | `246` | `11110110` | `366` | `F6` |
| A3 | `2`@10, `1010`@2, `A`@16 | `a * b * c` | `200` | `11001000` | `310` | `C8` |
| A4 | `100`@10, `4`@10, `5`@10 | `a / b / c` | `5` | `101` | `5` | `5` |
| A5 | `2`@10, `3`@10, `4`@10 | `(a + b) * c` | `20` | `10100` | `24` | `14` |
| A6 | `2`@10, `3`@10, `4`@10 | `a + b * c` | `14` | `1110` | `16` | `E` |

### Complement tests

1. Enter `11`, `5`, and `2` as Decimal inputs and select Decimal. Every active input should receive an individual card with 9's and 10's complements in two columns.
2. The card for `11` should show `9 - 1 = 8`, 9's complement `88`, and 10's complement `89`.
3. Enter `10.5` and `2.25`. The Decimal subtraction walkthrough should align them as `10.50` and `02.25`, then produce `08.25` with both methods.
4. Select Input #2 as `X` and Input #3 as `Y`. Both subtraction methods should calculate `5 - 2 = 3`.
5. Reverse the selectors to calculate `2 - 5`. Both methods should show a negative result and re-complementing.
6. Select Hexadecimal. Every active input should show 15's and 16's complements.
7. Enter a negative value. The complement output should clear and show a validation message.
8. Enter `0.333333` and `0.1`. The Decimal complement view should use `0.33333` and the subtraction walkthrough should complete without a repeating-representation error.

## Sample Outputs

Existing converter samples:

- [Decimal input](sample-outputs/decimal-input.png)
- [Fractional decimal input](sample-outputs/fractional-decimal-input.png)
- [Hexadecimal input](sample-outputs/hexadecimal-input.png)
- [Invalid input](sample-outputs/invalid-input.png)
- [Long fractional output](sample-outputs/long-fractional-output.png)

The application flowchart is shown below. It covers input validation, number-system conversion, PEMDAS expression evaluation, and arithmetic error handling. The complement workflow is documented immediately after the image because the original submitted PNG predates the complement feature.

![Number system converter flowchart](sample-outputs/Flowchart.png)

The current complement branch follows the same flowchart rules: select a radix tab, display each active input's `(r-1)` and `r` complements, select Minuend `X` and Subtrahend `Y`, align fixed-point digits to five fractional places, then process diminished-radix and radix subtraction with carry or negative re-complement handling. Standard symbols are used conceptually: terminators for start/end, parallelograms for input/output, rectangles for processes, and diamonds for decisions.

## How to Run

1. Open the project folder.
2. Open [app.html](app.html) in a modern browser.
3. Select a source base and enter values.
4. Read the four conversion results for each row.
5. Use the expression builder for arithmetic across inputs.
6. Select a radix tab to inspect every active input's complements.
7. Choose Minuend `X` and Subtrahend `Y` to run complement subtraction.

## Limitations and Notes

- Complement operations support non-negative fixed-point values. Whole and fractional digits are aligned with leading and trailing zeroes before calculation.
- Complement calculations use at most five fractional digits. Repeating representations are truncated at the fifth digit for fixed-point complement work; they are not rounded or rejected.
- Converter and expression arithmetic support exact fractions, but repeating target-base fractions are limited to 32 generated fractional digits and receive `...`.
- Collapsed result tiles show at most five fractional digits; clicking expands them to the generated value.
- Complement width uses the larger whole-number width plus the aligned fractional width of the selected operands.
- The complement viewer shows all active valid inputs; subtraction uses only selected `X` and `Y`.
- The theme follows the browser's color-scheme preference when the page loads and is not persisted.
- Clipboard copying depends on browser permission and Clipboard API support.
- The application uses client-side JavaScript only and has no server-side persistence.
