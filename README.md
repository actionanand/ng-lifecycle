# Angular Life Cycles

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 18.0.1.

## Cloning Guide

1.  Clone only the remote primary HEAD (default: origin/master)

```bash
git clone <url> --single-branch
```

2. Only specific branch

```bash
git clone <url> --branch <branch> --single-branch [<folder>]
```

```bash
git clone <url> --branch <branch>
```

3. Cloning repositories using degit

   - main branch is default.

```bash
npx degit github:user/repo#branch-name <folder-name>
```

4. Cloning this project with skeleton

```bash
git clone https://github.com/actionanand/ng-lifecycle.git --branch 1-skeleton angular-proj-name
```

```bash
npx degit github:actionanand/ng-lifecycle#1-skeleton angular-proj-name
```

## Automate using `Prettier`, `Es Lint` and `Husky`

1. Install the compatible node version

```bash
  nvm install v20.13.1
```

2. Install and Configure Prettier

   - Install prettier as below:

   ```bash
     yarn add prettier -D
   ```

   - Create a `.prettierrc` file and write down the format as below: - [online ref](https://prettier.io/docs/en/options.html)

   ```yml
   trailingComma: 'all'
   tabWidth: 2
   useTabs: false
   semi: true
   singleQuote: true
   bracketSpacing: true
   bracketSameLine: true
   arrowParens: 'avoid'
   printWidth: 120
   overrides:
     - files:
         - '*.js'
         - '*.jsx'
       options:
         bracketSpacing: true
         jsxSingleQuote: true
         semi: true
         singleQuote: true
         tabWidth: 2
         useTabs: false
     - files:
         - '*.ts'
       options:
         tabWidth: 2
   ```

   - Create a `.prettierignore` file and write as below(sample)

   ```gitignore
   # Ignore artifacts:
   build
   coverage
   e2e
   node_modules
   dist
   dest
   reports

   # Ignore files
   *.lock
   package-lock.json
   yarn.lock
   ```

3. Install `Es Lint`, if not installed

```bash
ng add @angular-eslint/schematics
```

if error comes, use the below command

```shell
ng add @angular-eslint/schematics@next
```

4. Configure pre-commit hooks

Pre-commit hooks are a nice way to run certain checks to ensure clean code. This can be used to format staged files if for some reason they weren’t automatically formatted during editing. [husky](https://github.com/typicode/husky) can be used to easily configure git hooks to prevent bad commits. We will use this along with [pretty-quick](https://github.com/azz/pretty-quick) to run Prettier on our changed files. Install these packages, along with [npm-run-all](https://github.com/mysticatea/npm-run-all), which will make it easier for us to run npm scripts:

```bash
yarn add husky pretty-quick npm-run-all -D
```

To configure the pre-commit hook, simply add a `precommit` npm script. We want to first run Prettier, then run TSLint on the formatted files. To make our scripts cleaner, I am using the npm-run-all package, which gives you two commands, `run-s` to run scripts in sequence, and `run-p` to run scripts in parallel:

```json
  "precommit": "run-s format:fix lint",
  "format:fix": "pretty-quick --staged",
  "format:check": "prettier --config ./.prettierrc --list-different \"src/{app,environments,assets}/**/*{.ts,.js,.json,.css,.scss}\"",
  "format:all": "prettier --config ./.prettierrc --write \"src/{app,environments,assets}/**/*{.ts,.js,.json,.css,.scss}\"",
  "lint": "ng lint",
```

5. Initialize husky

   - Run it once

   ```bash
     npm pkg set scripts.prepare="husky install"
     npm run prepare
   ```

   - Add a hook

   ```bash
     npx husky add .husky/pre-commit "yarn run precommit"
     npx husky add .husky/pre-commit "yarn test"
     git add .husky/pre-commit
   ```

   - Make a commit

   ```bash
     git commit -m "Keep calm and commit"
     # `yarn run precommit and yarn test` will run every time you commit
   ```

6. How to skip prettier format only in particular file

   1. JS

   ```js
   matrix(1, 0, 0, 0, 1, 0, 0, 0, 1);

   // prettier-ignore
   matrix(
       1, 0, 0,
       0, 1, 0,
       0, 0, 1
     )
   ```

   2. JSX

   ```jsx
   <div>
     {/* prettier-ignore */}
     <span     ugly  format=''   />
   </div>
   ```

   3. HTML

   ```html
   <!-- prettier-ignore -->
   <div         class="x"       >hello world</div            >

   <!-- prettier-ignore-attribute -->
   <div (mousedown)="       onStart    (    )         " (mouseup)="         onEnd      (    )         "></div>

   <!-- prettier-ignore-attribute (mouseup) -->
   <div (mousedown)="onStart()" (mouseup)="         onEnd      (    )         "></div>
   ```

   4. CSS

   ```css
   /* prettier-ignore */
   .my    ugly rule
     {
   
     }
   ```

   5. Markdown

   ```md
     <!-- prettier-ignore -->

   Do not format this
   ```

   6. YAML

   ```yml
   # prettier-ignore
   key  : value
     hello: world
   ```

   7. For more, please [check](https://prettier.io/docs/en/ignore.html)

## Resources

- [GitHub Actions for Angular](https://github.com/rodrigokamada/angular-github-actions)
- [Angular 16 - milestone release](https://github.com/actionanand/ng16-signal-milestone-release)

## afterNextRender and afterRender

Sometimes it’s necessary to use browser-only APIs to manually read or write the DOM. This can be challenging to do with the [lifecycle events](https://angular.io/guide/lifecycle-hooks#lifecycle-event-sequence) above, as they will also run during [server-side rendering and pre-rendering](https://angular.io/guide/glossary#server-side-rendering). For this purpose, Angular provides afterRender and afterNextRender. These functions can be used unconditionally, but will only have an effect on the browser. Both functions accept a callback that will run after the next [change detection](https://angular.io/guide/glossary#change-detection) cycle (**including any nested cycles**) has completed.

Both hooks must be invoked within the injection context (inside constructor) and can accept an injector if there’s a need to run them outside of this context:

```ts
@Component()
export class MyCmp {
  private injector = inject(Injector);

  onClick() {
    afterNextRender(
      () => {
        // Do something
      },
      { injector: this.injector },
    );
  }
}
```

These 2 hooks are right, **If you need to perform DOM manipulations after the view is fully rendered** and stable.

### afterNextRender Hook

The `afterNextRender` hook, albeit not the most descriptive name, takes a callback function that **runs once after the subsequent change detection cycle**. Typically, afterNextRender is ideal for performing one-time initializations, such as integrating third-party libraries or utilizing browser-specific APIs.

The `afterNextRender` callback **executes once per `tick`** and then destroys itself. This implies that while we're not restricted to calling it only once, each invocation ensures it runs just once after the subsequent tick.

```ts
@Component()
export class MyCmp {
  constructor() {
    afterNextRender(() => {
      console.log('run once');
    });

    setTimeout(() => {
      afterNextRender(() => {
        console.log('run once');
      });
    }, 5000);
  }
}
```

### afterRender Hook

Generally, if you need to manually read or write any layout data, such as size or location, you should use `afterRender`. The purpose of this function is to synchronise with the DOM and it is called after every change detection cycle that follows.

```ts
@Component({
  selector: 'my-cmp',
  template: `<span #content></span>`,
})
export class MyComponent {
  content = viewChild.required<ElementRef>('content');

  constructor() {
    afterRender(() => {
      console.log(this.content().nativeElement.scrollHeight);
    });
  }
}
```

### Managing Hook Phases

When employing `afterRender` or `afterNextRender`, you have the option to specify a phase, offering precise control over the sequencing of DOM operations. This capability allows you to arrange write operations before read operations, thereby minimizing [layout thrashing](https://web.dev/avoid-large-complex-layouts-and-layout-thrashing).

```ts
@Component({...})
export class ExampleComponent {
  private elementWidth = 0;
  private elementRef = inject(ElementRef);

  constructor() {
    const nativeElement = this.elementRef.nativeElement;

    // Write phase: Adjusting width.
    afterNextRender(() => {
      nativeElement.style.width = computeWidth();
    }, { phase: AfterRenderPhase.Write });

    // Read phase: Retrieving final width after adjustments.
    afterNextRender(() => {
      this.elementWidth = nativeElement.getBoundingClientRect().width;
    }, { phase: AfterRenderPhase.Read });
  }
}
```

    1. `EarlyRead`: This phase is ideal for reading any layout-affecting DOM properties and styles necessary for subsequent calculations. If possible, minimize usage of this phase, favoring the Write and Read phases.
    2. `MixedReadWrite`: This phase serves as the default option. It’s suitable for operations requiring both read and write access to layout-affecting properties and styles. However, it’s advisable to use the explicit Write and Read phases whenever possible.
    3. `Write`: Opt for this phase when setting layout-affecting DOM properties and styles.
    4. `Read`: Utilize this phase for reading layout-affecting DOM properties.

- Source - [Exploring Angular’s afterRender and afterNextRender Hooks](https://netbasal.com/exploring-angulars-afterrender-and-afternextrender-hooks-7133612a0287)
