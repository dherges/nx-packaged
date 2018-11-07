# NxPackaged

> Demonstrates ng-packagr alongside Nx Workspace.

Develop your libraries and applications in a monorepo.
Build distribution-ready binaries in Angular Package Format.

[![Packaging Angular Libraries](./docs/build-with-ng-packagr.gif)](http://recordit.co/Jw0inbop7f)

[![Renovate enabled](https://img.shields.io/badge/renovate-enabled-brightgreen.svg?style=flat-square)](https://renovateapp.com/)
[![CircleCI](https://img.shields.io/circleci/project/github/ng-packagr/nx-packaged/master.svg?style=flat-square&label=Circle%20CI)](https://circleci.com/gh/ng-packagr/nx-packaged)


## Transcript

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 1.5.0 using [Nrwl Nx](https://nrwl.io/nx).

The following transcript documents reproducible steps to set up this project.

#### Create Nx Worksapce

First, [create an Nx Workspace](https://nrwl.io/nx/guide-nx-workspace).
This repository was created with the sandbox install script:

```bash
$ curl -fsSL https://raw.githubusercontent.com/nrwl/nx/master/packages/install/install-next.sh | bash -s nx-packaged
```

Alternatively, you could convert an existing Angular CLI project to Nx Workspace.
Please see the Nx documentation how to do that.

#### Create an app

For show-casing and documenting your library, [create an Angular app](https://nrwl.io/nx/guide-nx-workspace#create-an-app) in the Nx Workspace.

```bash
$ ng generate app one-app
```

#### Create a library

Now, [let's create one library for reusable business code and a second library for reusable Angular building blocks](https://nrwl.io/nx/guide-nx-workspace#create-a-lib):

```bash
$ ng generate lib one-lib --nomodule
$ ng generate lib two-lib --ngmodule
```

#### Develop libraries

Let's generate a component in the Angular library:

```bash
$ ng generate component myButton --app=two-lib
```

We also need to export the component through an Angular module in `two-lib.module.ts`:

```ts
import { MyButtonComponent } from './my-button/my-button.component';

@NgModule({
  imports: [CommonModule],
  declarations: [MyButtonComponent],
  exports: [MyButtonComponent]
})
export class TwoLibModule {}
```

And let's also implement some very veeee-ry smart business code in `one-lib.ts`:

```ts
export class OneLib {

  public foo(): string {
    return "bar";
  }
}
```

#### Develop the app

Now, import the libraries with `@nx-packaged/one-lib` and `@nx-packaged/two-lib` in our application.

Add the `TwoLibModule` to `app.module.ts`:

```ts
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { TwoLibModule } from '@nx-packaged/two-lib';

@NgModule({
  imports: [ /* ... */ TwoLibModule],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

Use the `OneLib` class in `app.component.ts`:

```ts
import { Component, OnInit } from '@angular/core';
import { OneLib } from '@nx-packaged/one-lib';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {

  business: OneLib = new OneLib();
}
```

Wire up all the parts in `app.component.html`:

```html
<h2>Develop Angular libraries with ng-packagr in a monorepo</h2>

<p>Here is a reusable button component, implemented in a library:</p>
<app-my-button></app-my-button>

<p>Here is reusable business code from a library:</p>
<pre><code>{{ business.foo() }}</code></pre>
```

#### Serve the app

Run the app with the standard Angular CLI command:

```bash
$ ng serve
```

Open your browser at http://localhost:4200 and you will see "my-button works!" and "bar" printed on the screen.

#### Compile the libraries to Angular Package Format

Add ng-packagr to development dependencies:

```bash
$ yarn add --dev ng-packagr
```

For each library, add a `package.json` in the library folders.
Here is the example for the first library:

```json
{
  "$schema": "../../node_modules/ng-packagr/package.schema.json",
  "name": "@nx-packaged/one-lib",
  "version": "1.0.0",
  "ngPackage": {
    "lib": {
      "entryFile": "index.ts"
    },
    "dest": "@nx-packaged/one-lib"
  }
}
```

Add a build script to the `package.json` in the repository root folder:

```json
  "scripts": {
    "build:libs": "ng-packagr -p libs/one-lib/package.json"
  }
```

Now, build the library to its binary representation with the following command:

```bash
$ yarn build:libs
```

The binaries are written to `@nx-packaged/one-lib` in the repository root folder!


#### Optional: add a 'productive' app configuration

Add a configuration for Angular CLI to build the app from Angular Package Format bundles in the `@nx-packaged` folder.
The Nx Workspace configuration (by default) builds the app from TypeScript sources in `libs/*`.

This is a great way to verify that the application works with the distribution-ready binaries:

```bash
$ ng build --app one-app-from-packages --prod
```

However, it also forces you to re-build the library every time you change the sources!
During development you can now use `ng serve` for hot-reloading.
On a CI server and in build scripts, you can use the above `ng build` command to verify the binaries in Angular Package Format!

Relevant configuration in `.angular-cli.json`:

```json
    {
      "name": "one-app-from-packages",
      "root": "apps/one-app/src",
      "outDir": "dist/apps/one-app",
      "assets": [
        "assets",
        "favicon.ico"
      ],
      "index": "index.html",
      "main": "main.ts",
      "polyfills": "polyfills.ts",
      "test": "../../../test.js",
      "tsconfig": "../../../tsconfig.packages.json",
      "testTsconfig": "../../../tsconfig.spec.json",
      "prefix": "app",
      "styles": [
        "styles.css"
      ],
      "scripts": [],
      "environmentSource": "environments/environment.ts",
      "environments": {
        "dev": "environments/environment.ts",
        "prod": "environments/environment.prod.ts"
      }
    }
```
