# TypeScript Coding standard

A good coding standard helps the code quality and development productivity because best practices are built-in. Coding standard includes code analysis and code formatter. This is a document for setup and execute coding standard for TypeScript-based Angular and Ionic development.

## 1 Introduction

There are two tools used in maitaining TypeScript coding standard: [TSLint](https://github.com/palantir/tslint) and [Prettier](https://prettier.io/). TSLint is used for code analysis and prettier is used for code format. Both tools should be configured and used in both IDE and the automatic build process.

## 2 TSLint

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

## 3 Prettier

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

## 4 Configure Pre-commit Hooks

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
