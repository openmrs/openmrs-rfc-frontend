# Project title
- Start Date: 2020/04/23
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/24

## Decision, including impact on distributions
All frontend modules should use a common visual coding style to simplify
collaboration and streamline discussions.

Consequently, repositories in the `openmrs-esm-*` space should opt-in to use a
set of common files:

- `.editorconfig`
- `prettier.config.js`
- `.eslintrc.json`

Presumably, the following contents make sense.

For the `.editorconfig` file we propose to use:

```plain
root = true

[*]
indent_style = space
indent_size = 2
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
end_of_line = lf
max_line_length = null
```

For the `prettier.config.js` file we propose to use:

```js
module.exports = {
  printWidth: 120,
  singleQuote: true,
  trailingComma: 'all',
  bracketSpacing: true,
  parser: 'typescript',
  semi: true,
  jsxBracketSameLine: true,
};
```

For the `.eslintrc.json` file we propose to use:

```json
{
  "extends": [
    "ts-react-important-stuff",
    "plugin:prettier/recommended"
  ],
  "parser": "babel-eslint"
}
```

## Definition
A **visual coding style** defines how the syntax of a programming language is
expressed in terms of flexible layers, mostly optional parentheses, white space,
and line breaks.

The ability to make enhanced style assertions and verify certain coding patterns
is called **linting**. A **linter** is a tool to perform these verifications.

## Reason for decision
- Having a single visual coding style simplifies reading code across repositories
- Tooling enhanced formatting saves time
- Proven patterns for readability should be applied
- Editorconfig is a defacto standard with plugins for every IDE
- Prettier is the most popular TypeScript code formatting tool
- ESLint is the official linter for TypeScript and JavaScript

## Alternatives
Go on with the repositories to be as is. Consequently, different development
styles and different essential barriers for coding will enter and have to be
fought manually.

Another alternative is to not go with all three, but rather with only a subset.
Yet another option may be to use other tools. For instance, we could leverage
the deprecated TSLint instead of ESLint, too.

## Common practices (not enforced)
- The integration in Webpack or CI/CD pipelines (e.g., as pre-checks)
- Prepush hooks for validating code already locally
- Discuss if extensions of the available files should be put into the global definitions
