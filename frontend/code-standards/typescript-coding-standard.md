# TypeScript Coding standard

A good coding standard helps the code quality and development productivity because best practices are built-in. Coding standard includes code analysis and code formatter. This is a document for setup and execute coding standard for TypeScript-based Angular and Ionic development.

## 1 Introduction

The coding guidelines are defined in the next section. There are two tools used in maitaining TypeScript coding standard: [TSLint](https://github.com/palantir/tslint) and [Prettier](https://prettier.io/). TSLint is used for code analysis and prettier is used for code format. Both tools should be configured and used in both IDE and the automatic build process.

## 2 Guidelines

### 2.1 Names

1. Use PascalCase for type names.
1. Do not use "I" as a prefix for interface names.
1. Use PascalCase for enum values.
1. Use camelCase for function names.
1. Use camelCase for property names and local variables.
1. Do not use `_` as a prefix for private properties.
1. Use whole words in names when possible.

### 2.2 Types

1. Do not export types/functions unless you need to share it across multiple components.
1. Do not introduce new types/values to the global namespace.
1. Shared types should be defined in a `types.ts` file or in files in `models` directory.
1. Within a file, type definitions should come first.

### 2.3 Usage

1. Use `undefined`, don't use `null`.
1. Consider objects like Nodes, Symbols, etc. as immutable outside the component that created them. Do not change them.
1. Consider arrays as immutable by default after creation.
1. More than 2 related Boolean properties on a type should be turned into an Enum flag.
1. Try to use `ts.forEach`, `ts.map`, and `ts.filter` instead of loops, i.e., no `for..in`, when it is not strongly inconvenient.

### 2.4 Comments

1. Use JSDoc style comments for functions, interfaces, enums, and classes.

### 2.5 Diagnostic Messages

1. All strings visible to the user need to be localized.
1. Use a period at the end of a sentence.
1. Use indefinite articles for indefinite entities.
1. Definite entities should be named (this is for a variable name, type name, etc..).
1. When stating a rule, the subject should be in the singular (e.g. "An external module cannot..." instead of "External modules cannot...").
1. Use present tense.

### 2.6 Angular and others

The Google Angular team has [a style guide](https://angular.io/guide/styleguide) that has many rules applicable for other front end develpment. Some useful rules include naming, small functions, single responsible principle, filename and etc.

## 3 TS Compiler Options

First of all, set the following compiler options in `tsconfig.json` to enforce strict type checking and error detection.

```json
"strict": true,
"noUnusedLocals": true,
"noUnusedParameters": true,
"noImplicitReturns": true,
"noFallthroughCasesInSwitch": true,
"skipLibCheck": true,
```

In the above configuration, enabling `--strict` enables `--noImplicitAny`, `--noImplicitThis`, `--alwaysStrict`, `--strictNullChecks`, `--strictFunctionTypes` and `--strictPropertyInitialization`.

The `skipLibCheck` tells the compiler to skil checking for all declaraton files (`*.d.ts`). There was an error in an Ionic 3.9 declaration file.

## 4 TSLint

Both the Angular CLI tool `ng` and the Ionic CLI tool `ionic` generate a `tslint.json` file. First, install and use `tslint-angular` preset for code analyzer: `npm i -D tslint-angular`.

Then, disable code style checking rules. Update the `tslint.json` file to have the following content for an Angulr/Ionic project.

```json
{
  "extends": ["tslint-angular"],
  "rules": {
    "comment-format": false,
    "curly": false,
    "eofline": false,
    "import-spacing": false,
    "indent": false,
    "max-line-length": false,
    "no-trailing-whitespace": false,
    "one-line": false,
    "quotemark": false,
    "semicolon": false,
    "typedef-whitespace": false,
    "whitespace": false,
    "angular-whitespace": false,
    "max-classes-per-file": false
  }
}
```

For an Ionic project, write the npm run command as `"lint": "tslint -c tslint.json -p tsconfig.json",`. You also need to change the `ionic` generated component class names to follow the [Angular code style](https://angular.io/guide/styleguide).

## 5 Prettier

First, install the Prettier package using `npm i -D prettier`. Then create a configuration file named `.prettierrc` that has customized rules:

```json
{
  "bracketSpacing": true,
  "printWidth": 80,
  "tabWidth": 2,
  "trailingComma": "all",
  "semi": false,
  "singleQuote": true,
  "useTabs": false
}
```

To make it format files automatically, set `“editor.formatOnSave”: true` in VS Code after installing the Prettier extension.

## 6 Configure Pre-commit Hooks

Use [`husky`](https://github.com/typicode/husky), [`pretty-quick`](https://github.com/azz/pretty-quick) and [`npm-run-all`](https://github.com/mysticatea/npm-run-all) to run Prettier on changed files before commit.

Run `npm install npm-run-all husky pretty-quick -D` and add the following scripts to `package.json` that fix format and lint sequentially.

```json
"format:fix": "pretty-quick --staged",
"precommit": "run-s format:fix lint",
```

The `husky` version 1.0 may change the configuration as the following:

```json
{
  "format:fix": "pretty-quick --staged",
  "husky": {
    "hooks": {
      "pre-commit": "run-s format:fix lint",
      "pre-push": "run-p test e2e"
    }
  }
}
```
