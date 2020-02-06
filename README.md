# Introduction

This exercise is part of the TAS course at the HAN University of Applied Sciences. It helps you to understand how the DOM works and how if influences security aspects of a basic web application

# Prerequisites

* You need to have a Google account to get access to Firebase
* Node and npm installed

# Learning objectives

* Perform basic DOM operations in TypeScript like finding and modifying HTML elements
* Able to build a web application using HTML, CSS and TypeScript using npm 
* Able to start and build a web application using npm and packages like concurrently and lite
* Knows the difference between normal and development dependencies in a package.json file
* Able to add and modify scripts in an existing package.json file
* Able to use a linter to validate TypeScript code against standardized rules
* Knows the OWASP top 10 web security vulnerabilities
* Able to defend against typical web security vulnerabilities using TypeScript and the browser (leaving out things to do on a web server) 
* Able to deploy a web application to Firebase Hosting

# Steps

1. Create a new project

Run the following command to create a new project:

```npm create```

Accept defaults for the package name and the version name, add the introductionary text from this exercise as a description. Accept the default entry point. Leave the test command, keywords and git repository empty and add your own name as the author. Accept ISC as the default for the license and type ```yes``` when asked to confirm the setup. 

2. Add dependencies

To be able to modify the application while running we need a few dependencies:

``` npm install typescript --save-dev ```

``` npm install concurrently --save ```

``` npm install lite-server --save-dev ```

``` npm install eslint --save-dev ```

Modify the scripts section in the package.json file:

```javascript
"scripts": {
    "test": "echo 'we are live!'",
    "lint-prepare": "node_modules/.bin/eslint --init",
    "lint-all": "node_modules/.bin/eslint *.ts",
    "tsc": "node_modules/.bin/tsc",
    "tsc:w": "node_modules/.bin/tsc -w",
    "lite": "node_modules/.bin/lite-server",
    "start": "node_modules/.bin/concurrently \"npm run tsc:w\" \"npm run lite\" "
  }
```
3. Add an index.html file:

```html
<!DOCTYPE html>
<html>    
    <body>
        <h1>Fun with TypeScript</h1>
        <p id="index">Let's rock</p>
        <label>Voer hier een tekst in: </label> <input id="echo"/><button id="evalEcho">Evalueer input</button>
        <p id="output"></p>
        <iframe class="e2e-iframe-trusted-src" width="640" height="390" id="yt"></iframe>
        <script src="htmlmodifier.js"></script>
        <script src="index.js"> </script>
    </body>
</html>
```

Run ```npm run start```. Your browser will open the HTML file. Open the Developer Toolbar and notice that there is an error loading the js files, an HTTP status code 404 is shown. 

4. Before adding the index.ts (! notice the TypeScript extension) file, first add a file called _htmlmodifier.ts_ with the following content:

```typescript
interface IHTMLModifier {
    modify(): void;
}

interface IConstructor<T> {
    readonly prototype: T;

    new(...args: any[]): T;
}

class HTMLModifier {
    public static GetImplementations = (): Array<IConstructor<IHTMLModifier>> => HTMLModifier.implementations;

    public static register<T extends IConstructor<IHTMLModifier>>(ctor: T) {
        HTMLModifier.implementations.push(ctor);
        return ctor;
    }

    private static implementations: Array<IConstructor<IHTMLModifier>> = [];
}
```

This code can be used to find all classes implementing the IHTMLModifier interface (for now, none available).

Second, add a file called _index.ts_ with the following content:

```typescript
document.body.onload = () => {
    HTMLModifier.GetImplementations().forEach((Modifier) => {
        new Modifier().modify();
    });
};
```

Before running this code let's see if it adheres to the TypeScript standard:

``` npm run lint-all ```

This will probably fail due to a missing eslint configuration. Run ``` npm run lint-prepare``` and check the option "To check syntax, find problems, and enforce code style ". In this example we won't use modules so check the option "None of these". We're not using any framework (None of these) but we will use TypeScript (y) in the Browser. We will use a popular style guide: Google. We'll use JavaScript as a format for the config file. Accept the option to add extra dependencies (Y). 

If ESLint is all set up run:``` npm run lint-all ```.

The current code does not fully conform to Google's standards, for now we'll fix this by changing a few rules. Open the file .eslintrc.js and add change the rules section to:

```javascript
'rules': {
    "require-jsdoc": "off",
    "no-unused-vars": "off",
    "max-len": ["error", { "code": 120 }],
    "indent": ["error", 4],
    "new-cap": "off"
  },
```

5. The code we added in the previous steps can be used as a [decorator](https://www.typescriptlang.org/docs/handbook/decorators.html). To be able to actually use this code we have to enable this feature for the TypeScript compiler, this is possible using a compiler parameter but we're going to use a new file, create a file called tsconfig.json with the following content:

```javascript
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "target": "es2015",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019' or 'ESNEXT'. */
    "module": "esnext",                     /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */
    "strict": true,                           /* Enable all strict type-checking options. */
    "esModuleInterop": true,                  /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    "forceConsistentCasingInFileNames": true  /* Disallow inconsistently-cased references to the same file. */
  }
}
```

Decorators are like Annotations in the Java language like to know from the OOPD-course, for example 
```java
@Override
```
 
6. This exercise should help you to find security weaknesses in basic HTML/TypeScript applications but before we can discover these weaknesses and trying to solve them there is one minor detail to discuss. Notice the reference in the HTML-file to a file called index.js. But we do not have such a file! That's why we have the npm start script:

* It runs the TypeScript compiler that creates .js files from .ts files
* It starts a browser with the HTML-file referencing the .js files
* It monitors changes in the .ts files, runs the TypeScript compiler when necessary and refreshes the browser.

Run ```npm start``` to run the application. We're ready to investigate some security weaknesses!

7. Cross Site Scripting

8. Cross Origin Resource Sharing

9. Content Security Policy

7. Deploy on Firebase
