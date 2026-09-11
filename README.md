# Number System Converter & Calculator

A browser-based number system converter and arithmetic calculator for working with binary, octal, decimal, and hexadecimal values. It converts a value among all four representations and evaluates click-built expressions across the entered inputs, showing the result in every base along with a detailed, step-by-step solution. The application is implemented as a self-contained HTML file with embedded CSS and JavaScript.

## Features

- Convert any value among binary, octal, decimal, and hexadecimal in real time.
- Evaluate expressions containing addition, subtraction, multiplication, division, unary signs, and parentheses across the inputs, even when they use different bases.
- Apply standard operator precedence and left associativity.
- Mixed-base arithmetic: each input keeps its own base and is converted to a common representation before the operation is applied.
- Exact arithmetic using `BigInt` rational values, with no floating-point rounding error.
- A detailed step-by-step solution: a positional-notation breakdown of each conversion to decimal followed by the final calculation.
- The arithmetic result is displayed in all four number systems.
- Clear handling of invalid inputs and division by zero.

## System Requirements

### Hardware

- Any computer, tablet, or phone capable of running a modern web browser.
- No special hardware or calculator is required.

### Software

- A modern browser with support for:
  - JavaScript ES2020 or later
  - `BigInt`
  - `String.prototype.startsWith`
  - DOM event handling
  - Clipboard API for copying results when permitted by the browser
- Examples of compatible browsers include current versions of Chrome, Edge, Firefox, and Safari.
- No server, database, package installation, or internet connection is required.
- To run the program, open `app.html` in a browser.

## Supported Number Systems

| Number system | Base | Accepted digits |
|---|---:|---|
| Binary | 2 | `0-1` |
| Octal | 8 | `0-7` |
| Decimal | 10 | `0-9` |
| Hexadecimal | 16 | `0-9`, `A-F`, or `a-f` |

A leading minus sign and one radix point (`.`) are accepted for every base. Examples of valid forms include `10.5`, `1010.1`, `2A.8`, `10.`, and `.5`. Leading and trailing spaces are removed before validation and conversion.

## Algorithm / Pseudocode

The application converts an input in two stages: the source value is converted to an exact rational value using two `BigInt` values, then that rational value is converted to each target base.

### Main conversion algorithm

```text
START

Create a minimum of three input rows.

When the user types a value or changes its selected base:
    Read the input text and selected source base.
    Remove leading and trailing whitespace.

    IF the input is empty:
        Clear the error message and all result fields.
        STOP processing this row.

    Validate the input against the selected base:
        Binary      -> optional '-' followed by digits 0 or 1
        Octal       -> optional '-' followed by digits 0 through 7
        Decimal     -> optional '-' followed by digits 0 through 9
        Hexadecimal -> optional '-' followed by digits 0 through 9 or A through F

    IF validation fails:
        Mark the input as invalid.
        Display the appropriate error message.
        Clear all result fields.
        STOP processing this row.

    Convert the valid source string to an exact rational value:
        Handle a leading '-' separately.
        Split the input at the radix point.
        Join the whole and fractional digits.
        Set numerator = 0.
        FOR each joined digit from left to right:
            Convert the digit to its numeric value.
            numerator = numerator * source base + digit value.
        Set denominator = source base ^ number of fractional digits.
        Apply the negative sign to the numerator if necessary.

    For each target base in [2, 8, 10, 16]:
        Convert the rational value to the target base:
            Separate the sign and use the absolute numerator.
            Convert numerator / denominator to the integer part
            using repeated division and remainders.
            Set remainder = numerator modulo denominator.
            WHILE remainder is not zero and fewer than 32 fraction digits exist:
                Multiply remainder by the target base.
                The quotient is the next fraction digit.
                Keep the new remainder after division.
            Append "..." when the fraction is still repeating after 32 digits.
            Restore the negative sign if necessary.
        Display the converted value.

END
```

### Input-row management algorithm

```text
When Add Input is clicked:
    Create a new input row.
    Add it to the page.
    Renumber every row.

When a Remove button is clicked:
    IF there are only three rows:
        Do nothing.
    ELSE:
        Remove the selected row.
        Renumber every row.

When Clear Values is clicked:
    Empty every input field.
    Clear every error and result field.

When a result is clicked:
    Expand the result to its full generated value.
    Copy the full generated value to the clipboard.
    Collapse it again if clicked a second time.
    Briefly display "Copied" when copying succeeds.
```

### Expression evaluation algorithm

The arithmetic calculator reuses the same conversion inputs. Non-empty inputs are referenced in order as `a`, `b`, `c`, and so on. The expression builder creates the expression using buttons, so the user does not need to type operators or parentheses. Each value is converted to an exact rational before the expression is evaluated and the result is reported in all four bases.

```text
When an input value changes:
    Refresh the available variable buttons for the non-empty rows.
    Recalculate the current expression automatically.

When an input-variable or operator button is clicked:
    Add its token to the read-only expression display.
    Enable only buttons that are valid for the current expression state.
    Recalculate automatically when the expression is complete.

When Backspace or Clear is clicked:
    Remove the last token or clear the expression.
    Recalculate automatically.

When an expression is evaluated:
    Collect every non-empty input row in order.
    FOR each collected input:
        Validate the value against its selected base.
        IF validation fails:
            Show an error naming the input number.
            STOP.
        Convert the value to an exact rational (numerator, denominator).

    IF fewer than two valid values were collected:
        Show a message asking for at least two values.
        STOP.

    Tokenize the expression and parse it recursively:
        Parentheses and unary signs are handled first.
        Multiplication and division are handled next, left to right.
        Addition and subtraction are handled last, left to right.
    IF the expression is malformed, references an empty input, or divides by zero:
        Show a descriptive arithmetic error.
        STOP.

    Build the expression from the original inputs and their base subscripts.

    Build the step-by-step solution:
        FOR each input:
            IF it is decimal, note "Already in decimal".
            ELSE show a positional-notation breakdown:
                (digit x base^position) + ... on one line,
                the positional values on the next line,
                and the decimal value in its own column.
        Show the final calculation using the decimal values.

    Display the result in bases 2, 8, 10, and 16.
```

## Flowchart

```mermaid
flowchart TD
    A([Open app]) --> B[Enter values and choose their bases]
    B --> C{Are the values valid?}
    C -->|No| D[Show input error]
    D --> B
    C -->|Yes| E[Convert each value and show all four bases]
    E --> F[Build an expression with the buttons]
    F --> G{Expression complete?}
    G -->|No| F
    G -->|Yes| H[Evaluate with parentheses and operator precedence]
    H --> I{Arithmetic error?}
    I -->|Yes| J[Show arithmetic error]
    J --> F
    I -->|No| K[Show expression, solution, and result in all four bases]
```

The `Add Input`, `Remove`, `Clear Values`, and theme controls operate independently of the expression path. Removing a row uses a brief slide-and-fade animation, and the application always keeps at least three rows. Non-empty rows are exposed as variables in order: `a`, `b`, `c`, and so on. The expression builder recalculates automatically after each valid completed expression.

## Program Implementation

### File structure

```text
number-system-converter/
├── app.html
└── README.md
```

### User interface

The interface is contained in `app.html` and includes:

- A responsive converter workspace.
- Student information fields for the name and course.
- A light/dark theme toggle.
- A subtle transparent grid background.
- Three input rows displayed initially.
- Base selector buttons: `BIN`, `OCT`, `DEC`, and `HEX`.
- Four result fields per row.
- `Add Input`, `Remove`, and `Clear Values` controls.
- A click-only expression builder with input-variable, operator, parenthesis, Backspace, and Clear buttons.
- A result panel showing the arithmetic expression, a step-by-step solution table, and the result in all four bases.

### Validation implementation

The `BASE_INFO` object defines the accepted pattern for each base:

```javascript
const BASE_INFO = {
    2:  { name: "Binary",      pattern: /^-?(?:[01]+(?:\.[01]*)?|\.[01]+)$/i },
    8:  { name: "Octal",       pattern: /^-?(?:[0-7]+(?:\.[0-7]*)?|\.[0-7]+)$/i },
    10: { name: "Decimal",     pattern: /^-?(?:[0-9]+(?:\.[0-9]*)?|\.[0-9]+)$/i },
    16: { name: "Hexadecimal", pattern: /^-?(?:[0-9A-Fa-f]+(?:\.[0-9A-Fa-f]*)?|\.[0-9A-Fa-f]+)$/i }
};
```

The `validateInput()` function rejects characters that are not legal for the selected base. Empty input is treated as a cleared row rather than an error.

### Conversion implementation

- `baseStringToRational(rawValue, base)` converts a string from its selected base into an exact `{ numerator, denominator }` rational value.
- `rationalToBaseString(rational, base)` converts the rational value using integer division and repeated multiplication of the remainder.
- `BigInt` is used instead of JavaScript `Number`, allowing exact integer and finite-fraction conversion beyond the normal safe integer limit.
- Results show at most five digits after the radix point in their normal collapsed state.
- Clicking a result expands it to the full generated value. Repeating output is limited to 32 digits and ends with `...`.
- The `DIGITS` string, `0123456789ABCDEF`, supplies output symbols for all supported bases.
- `updateRow()` coordinates validation, conversion, error display, and result rendering.

### Arithmetic implementation

- The arithmetic feature operates on the same conversion input rows, so each operand keeps its own base.
- `collectOperands()` reads every non-empty row in order, validates it, and stores its exact rational value.
- `computeRational(a, b, op)` performs one operation on two rational values; addition, subtraction, multiplication, and division are each expressed as exact fraction arithmetic. Division by zero returns a null result that is reported to the user.
- `reduceRational()` divides the numerator and denominator by their greatest common divisor (`gcd()`) so results stay in lowest terms.
- `tokenizeExpression()` recognizes variables, operators, and parentheses while rejecting unsupported characters.
- `evaluateExpression()` uses recursive-descent parsing: unary signs and parentheses are handled first, multiplication and division next, and addition and subtraction last. Operators at the same precedence are evaluated left to right.
- `refreshOperandButtons()` and `updateBuilderButtonStates()` provide the click-only variable builder and prevent invalid next tokens.
- `performArithmetic()` runs automatically after expression-builder and input changes, then renders the expression, solution table, and four-base result tiles.
- `conversionBreakdown()` builds the positional-notation expansion for a value, for example `(4 x 8^2) + (2 x 8^1) + (1 x 8^0)` with the positional values on the line below. `superscript()` renders the exponents as Unicode superscripts, including negative exponents for fractional digits.
- The expression and result use the base number itself as a subscript, for example `1011(2) + 123(10) = 134(10)`.
- `performArithmetic()` coordinates collection, validation, computation, expression rendering, the solution table, and the four-base result tiles.

### Event handling

The program uses event delegation on the input container. This allows input, base-selection, result-copy, and remove actions to work for rows created after the initial page load. The expression builder uses delegated handlers for variable, operator, parenthesis, Backspace, and Clear buttons. Input and builder changes trigger automatic recalculation.

## Test Cases

The following cases can be tested directly in a browser by entering a value, selecting its source base, and checking all four result fields.

| Test | Input | Source base | Expected binary | Expected octal | Expected decimal | Expected hexadecimal | Result |
|---:|---|---:|---|---|---:|---|---|
| 1 | `42` | 10 | `101010` | `52` | `42` | `2A` | Pass |
| 2 | `101010` | 2 | `101010` | `52` | `42` | `2A` | Pass |
| 3 | `52` | 8 | `101010` | `52` | `42` | `2A` | Pass |
| 4 | `2A` | 16 | `101010` | `52` | `42` | `2A` | Pass |
| 5 | `-15` | 10 | `-1111` | `-17` | `-15` | `-F` | Pass |
| 6 | `0` | 10 | `0` | `0` | `0` | `0` | Pass |
| 7 | `10.5` | 10 | `1010.1` | `12.4` | `10.5` | `A.8` | Pass; fractional input |
| 8 | `-2A.8` | 16 | `-101010.1` | `-52.4` | `-42.5` | `-2A.8` | Pass; negative fraction |
| 9 | `10201` | 2 | Blank | Blank | Blank | Blank | Error shown: invalid binary number; results are cleared |
| 10 | `1G` | 16 | Blank | Blank | Blank | Blank | Error shown: invalid hexadecimal number; results are cleared |

The invalid-input cases produce blank result fields, not the text `Not generated`.

The large-integer behavior is covered by the implementation because all arithmetic uses `BigInt`; it can also be checked by entering any integer larger than JavaScript's safe integer limit.

The main UI behavior is covered during normal testing: the page starts with three rows, `Add Input` adds a row, `Remove` animates and removes a row only when more than three exist, `Clear Values` clears all rows, and clicking a populated result expands and copies it.

### Arithmetic test cases

The following cases can be tested by entering the listed values in the input rows, selecting each source base, and building the listed expression with the expression buttons. Results update automatically when the expression is complete.

| Test | Inputs (value @ base) | Expression | Expected decimal result | Expected binary | Expected octal | Expected hexadecimal | Result |
|---:|---|---|---:|---|---|---|---|
| A1 | `1011`@2, `123`@10 | `a + b` | `134` | `10000110` | `206` | `86` | Pass; mixed-base addition |
| A2 | `10`@10, `20`@10, `30`@10 | `a + b + c` | `60` | `111100` | `74` | `3C` | Pass; three-input expression |
| A3 | `FF`@16, `1`@10, `10`@8 | `a - b - c` | `246` | `11110110` | `366` | `F6` | Pass; left-associative subtraction |
| A4 | `2`@10, `1010`@2, `A`@16 | `a * b * c` | `200` | `11001000` | `310` | `C8` | Pass; mixed-base multiplication |
| A5 | `100`@10, `4`@10, `5`@10 | `a / b / c` | `5` | `101` | `5` | `5` | Pass; left-associative division |
| A6 | `10`@10, `4`@10 | `a / b` | `2.5` | `10.1` | `2.4` | `2.8` | Pass; exact fractional result |
| A7 | `5`@10, `0`@10 | `a / b` | — | — | — | — | Error shown: cannot divide by zero |
| A8 | `1011`@2 only | `a + b` | — | — | — | — | Error shown: variable `b` has no matching input |
| A9 | `2`@10, `3`@10, `4`@10 | `(a + b) * c` | `20` | `10100` | `24` | `14` | Pass; parentheses override precedence |
| A10 | `2`@10, `3`@10, `4`@10 | `a + b * c` | `14` | `1110` | `16` | `E` | Pass; multiplication precedes addition |

The step-by-step solution for each case shows the positional-notation breakdown of every non-decimal input and the final calculation using the converted decimal values.

## Sample Output

The following screenshots show the application's submitted sample outputs.

### Sample 1: Decimal input

![Decimal input sample](sample-outputs/decimal-input.png)

The screenshot shows three decimal inputs: `2303`, `9480`, and `12503`. The application displays their binary, octal, decimal, and hexadecimal equivalents.

### Sample 2: Fractional decimal input

![Fractional decimal input sample](sample-outputs/fractional-decimal-input.png)

The screenshot shows decimal fractional inputs: `97.31`, `53.87`, and `17340.2`. Results with longer fractional representations are shortened with `...` in the collapsed display.

### Sample 3: Hexadecimal input

![Hexadecimal input sample](sample-outputs/hexadecimal-input.png)

The screenshot shows hexadecimal inputs: `41AEF`, `21CB02`, and `FF31`. The corresponding decimal results shown are `269039`, `2214658`, and `65329`.

### Sample 4: Invalid input

![Invalid input sample](sample-outputs/invalid-input.png)

The screenshot shows invalid values entered with different source bases: Binary `100200194`, Octal `811.478`, and Decimal `AE.7612`. Each row displays the appropriate validation message and leaves the result fields blank.

### Sample 5: Long fractional output

![Long fractional output sample](sample-outputs/long-fractional-output.png)

The screenshot shows long fractional values entered using different source bases: decimal `1.157486`, octal `347.021`, and hexadecimal `17FA3.11`. Results that exceed five fractional digits are shortened with `...` in the collapsed display, while shorter results remain fully visible.

## How to Run

1. Open the project folder.
2. Open `app.html` in a modern web browser.
3. Select the source base for an input row.
4. Enter a valid value.
5. Read the four generated representations.
6. Click a result to copy it when needed.

To use the arithmetic calculator:

1. Enter two or more values in the input rows, each with its own base.
2. Click the variable buttons (`a`, `b`, `c`, and so on) in the desired order.
3. Click operator and parenthesis buttons to build the expression. Use Backspace or Clear to correct it.
4. Read the automatically updated expression, step-by-step solution, and result shown in all four bases.

## Limitations and Notes

- Fractional values are supported using exact rational arithmetic. Collapsed results show up to five fractional digits and use `...` when more digits exist.
- Clicking a result expands it to the full generated value, up to 32 fractional digits; repeating values end with `...`.
- The expression evaluator follows standard precedence: parentheses and unary signs, then multiplication and division, then addition and subtraction. Operators at the same precedence are left associative.
- Arithmetic uses the same exact `BigInt` rational arithmetic as the converter, so results are exact; a repeating result (such as `1 / 3`) is shown to 32 fractional digits ending with `...`.
- Division by zero, malformed expressions, and references to unavailable variables are reported as arithmetic errors and produce no result. At least two valid inputs are required to calculate an expression.
- Prefixes such as `0b`, `0o`, and `0x` are not accepted because the validator expects digits only, with an optional leading minus sign.
- Theme selection follows the browser's current color-scheme preference when the page loads; it is not persisted after the page is closed.
- Clipboard copying depends on browser permission and support for `navigator.clipboard`.
