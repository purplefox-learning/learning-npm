# My Awesome Greeter

#### Master Reference
I learn this from https://itnext.io/step-by-step-building-and-publishing-an-npm-typescript-package-44fe7164964c.


#### Start from Scratch

`npm init -y` creates package.json

`npm install --save-dev typescript` so that the following config is added to package.json
"devDependencies": {
  typescript": "^3.2.1"
}

create tsconfig.json in the project root dir
* outDir is where javascript will get compiled to, in this case `lib`
* includde is where all the source files are, in this case `src`
```
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./lib",
    "strict": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "**/__tests__/*"]
}
```

create a src dir, create the first source file `src/index.ts`, take note the `` around Hello is NOT '' !!!
```
  export const Greeter = (name: string) => `Hello ${name}`;
```

add `"build": "tsc"` to the `scripts` block of package.json so that it now reads
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "tsc"
  },

run `npm run build`!

we will see a new `lib` folder in the root with the compiled code and type definitions.
we also see a new `node_modules` folder.

since both folders are auto generated, we ignore the top level lib dir, and node_modules in every possible dir in git
`echo "/lib" >> .gitignore`
`echo "node_modules" >> .gitignore`


#### Format and Lint

add formatting and linting, but only as a devDependency
`npm install --save-dev prettier tslint tslint-config-prettier`

in the project root, add a tslint.json
{
   "extends": ["tslint:recommended", "tslint-config-prettier"]
}

in the project root, add a .prettierrc
{
  "printWidth": 120,
  "trailingComma": "all",
  "singleQuote": true
}

update package.json to include both lint and format scripts under the `scripts` block
"format": "prettier --write \"src/**/*.ts\" \"src/**/*.js\"",
"lint": "tslint -p tsconfig.json"

let's test the lint and prettier
`npm run lint`
`npm run format`

add `files` block to package.json to whitelist the files/folders we want to publish to npm. opposite to git, we are not interested to publish src dir to npm
  "files": [
    "lib/**/*"
  ]


#### Unit Test

add a jest, a testing framework
`npm install --save-dev jest ts-jest @types/jest`

in the project root, add a jestconfig.json
{
  "transform": {
    "^.+\\.(t|j)sx?$": "ts-jest"
  },
  "testRegex": "(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$",
  "moduleFileExtensions": ["ts", "tsx", "js", "jsx", "json", "node"]
}

in package.json, replace the `test` of `scripts` block with 
`"test": "jest --config jestconfig.json",`

add a new testcase in src/__tests__ dir. the testcase file name should end with test.ts, for example Greeter.test.ts
import { Greeter } from '../index';
test('My Greeter', () => {
  expect(Greeter('Carl')).toBe('Hello Carl');
});

now test it `npm test`


#### Build, Package and Publish

add all these to the `scripts` block in package.json file
"prepare" : "npm run build"
"prepublishOnly" : "npm test && npm run lint"
"preversion" : "npm run lint"
"version" : "npm run format && git add -A src"
"postversion" : "git push && git push --tags"

a brief explanation on purposes of these scripts
* prepare will run both BEFORE the package is packed and published, and on local npm install. Perfect for running building the code. Add this script to package.json
* prepublishOnly will run BEFORE prepare and ONLY on npm publish
* preversion will run before bumping a new package version. To be extra sure that we’re not bumping a version with bad code, why not run lint here as well?
* version will run after a new version has been bumped. If your package has a git repository, like in our case. A commit and a new version-tag will be made every time you bump a new version. This command will run BEFORE the commit is made. One idea is to run the formatter here and so no ugly code will pass into the new version:
* postversion will run after the commit has been made. A perfect place for pushing the commit as well as the tag.

make a few more changes to package.json
   "main": "lib/index.js",
   "types": "lib/index.d.ts",

`npm login` since i already created an account at npmjs.com

`npm publish` to publish our package to npm.com
