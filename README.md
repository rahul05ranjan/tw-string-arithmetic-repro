# Repro: math-operator whitespace formatting reaches inside quoted strings

```sh
npm install
npm run build
# then look at the generated .content-\[\'\(min\(1\,2\)\\'\] rule in out.css
```

`content-['(min(1,2))']` compiles to `content: "(min(1, 2))"` — a space is inserted
after the comma **inside the quoted string**, changing the text the element
renders. Same for operators: `content-['(calc(1px+2px))']` becomes
`"(calc(1px + 2px))"`.

Control case `content-['(1+2)']` is untouched (no math function name present).

Cause (tailwindcss v4, `packages/tailwindcss/src/utils/decode-arbitrary-value.ts`
calls `addWhitespaceAroundMathOperators` from
`packages/tailwindcss/src/utils/math-operators.ts`): that pass walks the
serialized value character by character and, when it sees a `(` preceded by a
math function name, formats commas/operators until the matching `)`. It never
skips over quoted strings the way `segment()` and `isValidArbitrary()` in the
same folder do, so a math-function-shaped call that only exists as *text inside
a string* still gets formatted.
