# Ionic Test

Ionic Framework doesn't provide good testing utilities. Currently (2018/05) there are two options: the [Ionic test sample project](https://github.com/ionic-team/ionic-unit-testing-example) and the [clicker seed project](https://github.com/lathonez/clicker). The Ionic test sample is preferred because Ionic 4 is most likely based on it.

## 1. TDD Process

Following the TDD philosophy, the recommended development process is:

- write an e2e test for a specific requirement.
- the e2e test fails
- decide what functionality needs pass the e2e test.
- write a unit test for the functionality.
- the unit test fails.
- implement the code to satisfy the unit test.
- pass the unit test, revise the code if it fails.
- pass the e2e test, add/revise functionality if it fails.

The failure steps of both e2e and unit test are necessary to make sure two things: 1) the test is writting correctly, and 2) force to work within the parameters of the test.

To start a project, first come with a list of requirements and corresponding e2e tests. Then prioritize the requirements and e2e tests to make a minimum viable product (MVP) ASAP.

The key difference between a unit test and an E2E test is that a unit test will test code, whereas an E2E will test behaviour. An e2e test tests the page contents that include the current page and the navigated pages. A unit test only checks the logic of the current page. Therefore starting from the homepage, every e2e test tests the current page content and navigation to child pages. Each page's unit test tests the component logic such as componentent creation and setting member variables from the navigator's parameters.

## 2. Configure for Testing

The ["Testing in Ionic" blog](https://leifwells.github.io/2017/08/27/testing-in-ionic-configure-existing-projects-for-testing/) gives step-by-step configuration instructions. The following is a simplified version.

### Step 1: Install Required NPM Packages

Use the following command to install `karma`, `jasmine`, `protractor` and other utilities.

`npm install --save-dev angular2-template-loader html-loader jasmine jasmine-spec-reporter karma karma-chrome-launcher karma-jasmine karma-jasmine-html-reporter karma-sourcemap-loader karma-webpack karma-coverage-istanbul-reporter istanbul-instrumenter-loader null-loader protractor ts-loader@v3.5.0 ts-node @types/jasmine @types/node`

Use `ts-loader` 3.x for `webpack` 3.x -- Angular 5 and older uses this version.

### Step 2: Add the Configuration Files

Copy the `test-config` folder (with 5 files) from the [Ionic test sample project](https://github.com/ionic-team/ionic-unit-testing-example) to your project root.

The `karma-config.js` is the Karma configuration file that uses `webpack.test.js`, `karma-test-shim.js`. The `protractor.conf.js` is the Protractor configuration file that specifies the test file path, for example, `'../e2e/**/*.e2e-spec.ts'` means all files with a postfix of `.e2e-spec.ts` in the parent's `e2e` folder.

The `mocks-ionic.js` file is a collection of classes that mock `ionic-angular` classes.

### Step 3: Add Command-line Scripts

Add the following commands to `package.json`:

```json
"test": "karma start ./test-config/karma.conf.js",
"test-ci": "karma start ./test-config/karma.conf.js --single-run",
"test-coverage": "karma start ./test-config/karma.conf.js --coverage",
"e2e": "npm run e2e-update && npm run e2e-test",
"e2e-test": "protractor ./test-config/protractor.conf.js",
"e2e-update": "webdriver-manager update --standalone false --gecko false"
```

### Step 4: Add e2e Testing

Copy the `e2e` folder from the [Ionic test sample project](https://github.com/ionic-team/ionic-unit-testing-example) to your project root.

The `tsconfig.json` is a compiler configuration file. The `app.e2e-spec.ts` is an example test file and the `app.po.ts` is an example page object file that provides test utilities.

### Step 5: Add Your Unit and e2e tests

Unit tests have a postfix of `.spec.ts` and are stored with their corresponding components or class/funtion files. e2e tests have a postfix of `.e2e-spec.ts` and are put in `e2e` folder at the project root.

## 3 Run Tests

Run `ionic serve` before run the e2e test, otherwise Protractor cannot connect to the pages.
