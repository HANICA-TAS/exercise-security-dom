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
    "new-cap": "off",
    "no-eval": "error"
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

7. When you don't use a framework and want to use TypeScript to modify HTML and listen to events we need to use the DOM: Document Object Model. The DOM provides methods to read and modify the document (an object referring to the current HTML document) like finding elements and adding elements. Open the documentation about the [Document](https://developer.mozilla.org/en-US/docs/Web/API/Document) class and the [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element) class in your browser. Use this documentation to:

    * Create a new file called innerhtmlmodifier.ts
    * Create a new class called InnerHTMLModifier that implements the IHTMLModifier interface
    * Add a line before the class declaration to decorate it: ``` @HTMLModifier.register ```
    * Implement the function _modify_  to find the element with id "index" and change the innerHTML property to the value "Changed!"
    * Add the script-reference to the HTML file
    
    Sometimes the browser does not refresh when a new file is added, in that case use Ctrl^C to end the current run and run ``` npm run start``` again. 

    When your code works the page show the test "Changed!" just below the document title. In the next steps you will use the DOM to do other modifications, all having certain risks. 
    
8. Suppose you're working at StackOverflow, a site that enables developers to help each other with code issues. The current HTML has a small input box that might be used to enter code. 

    Add a new class OutputModifier in a file outputmodifier.ts with the same decorator like in the previous step. Implement the _modify_ function to find the button with id "evalEcho" and listen to the click event. When the button is clicked look up the element with id "output" and change the innerHTML property with the value from the input box  (which has the id "echo"). 
    
    This simulates the behaviour of adding a question about a certain code issue, save it to any database and reloads the screen with the question showing the question and the code. 
    
    Try typing some text into the edit box and click on the button, your text should be displayed just above the big square. 
     
    JavaScript has a function called _eval_ that can be used to execute a text (string) as JavaScript code. Modify your OutputModifier to change the innerHTML property to the execution of the eval function that takes the value from the inbox box as a parameter. 
    
    Try typing something like ``` alert("hi!") ``` in the box and click the button. Instead of displaying the text it gets executed as code. Some people like this dynamic feature but you can never trust the user's input and you should also be careful with carelessly displaying data from other external sources.
    
    Take-aways:
    * Eval is evil, do not use this feature. The linter will warn you for this, see for yourself: ``` npm run lint-all ```
    * Never trust use input and be careful displaying data from other external sources. You might ending up executing code that can harm your application or your user, this is called Cross Site Scripting (XSS). 
    
    Visit the [OWASP Top 10](https://owasp.org/www-project-top-ten/) site and read the section about XSS. 

9. Cross Origin Resource Sharing

10. Content Security Policy

11. Deploy on Firebase
