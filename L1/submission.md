### Scenario

You're assigned to a project that involves implementing a critical feature using a branching strategy inspired by GitFlow. Each developer creates feature branches, works on their assigned tasks, and then submits pull-requests for code review.

### Problem

You've been assigned to a project that involves enhancing a critical feature for a web application. The team places a strong emphasis on the pull-request workflow, with a focus on code reviews, merge conflict resolution, and the recent integration of CI/CD. As you navigate through the development task, you encounter challenges such as feedback during code reviews and discussions on effective merge conflict resolution. The team looks to you to demonstrate your understanding of these challenges and your ability to adapt to the added complexity of CI/CD integration.

### Application context

Before I provide my solution to L1, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

---
# Handling Code review feedback

### To understand `handle code review feedback` lets understand what the current proposed changes

Currently the `freedraw` section (where user can draw without needing to make pairs) uses a plain html canvas which limits user creativity.

The current change proposes using the [`tldraw`](https://www.npmjs.com/package/tldraw) library which allows better user experience.

##### File changes submitted

`frontend/src/components/TldCanvas.tsx` (file added)

```tsx
import { FunctionComponent } from "react";
import { Tldraw } from "tldraw";
import "tldraw/tldraw.css";

const TldCanvas: FunctionComponent = () => {
  return (
    <div
      style={{
        position: "fixed",
        top: 70,
        right: 0,
        bottom: 0,
        left: 0,
      }}
    >
      <Tldraw />
    </div>
  );
};

export default TldCanvas;
```

`frontend/src/pages/FreeDrawCanvas.tsx` (made changes)

```tsx
import { FunctionComponent } from "react";
import Navbar from "../components/Navbar";
import TldCanvas from "../components/TldCanvas";

const FreeDrawCanvas: FunctionComponent = () => {
  return (
    <>
    <Navbar />
    <TldCanvas />
  </>
  )
};

export default FreeDrawCanvas;
```
#### reviewed code

`frontend/src/components/TldCanvas.tsx`

```tsx
import { FunctionComponent } from "react";
import { Tldraw } from "tldraw";
import "tldraw/tldraw.css";

const TldCanvas: FunctionComponent = () => {
  return (
    <div
	  // consider placing the styles within the .css files to improve readability
      style={{
        position: "fixed",
        top: 70,
        right: 0,
        bottom: 0,
        left: 0,
      }}
    >
      <Tldraw />
    </div>
  );
};

export default TldCanvas;
```

`frontend/src/pages/FreeDrawCanvas.tsx` 

```tsx
import { FunctionComponent } from "react";
import Navbar from "../components/Navbar";
import TldCanvas from "../components/TldCanvas";

const FreeDrawCanvas: FunctionComponent = () => {
  // improve code formatting here, proper indentation is not used
  return (
    <>
    <Navbar />
    <TldCanvas />
  </>
  )
};

export default FreeDrawCanvas;
```

#### updated code

`frontend/src/components/TldCanvas.tsx`

```tsx
import { FunctionComponent } from "react";
import { Tldraw } from "tldraw";
import "tldraw/tldraw.css";
import "../styles/tld-styles";
const TldCanvas: FunctionComponent = () => {
  return (
    <div className="tld-canvas">
      <Tldraw />
    </div>
  );
};

export default TldCanvas;
```

`frontend/src/styles/tld-styles.css` 

```css
.tld-canvas {
	position: "fixed";
    top: 70;
	right: 0;
	bottom: 0;
	left: 0;
}
```

`frontend/src/pages/FreeDrawCanvas.tsx` 

```tsx
import { FunctionComponent } from "react";
import Navbar from "../components/Navbar";
import TldCanvas from "../components/TldCanvas";

const FreeDrawCanvas: FunctionComponent = () => {
  return (
    <>
	    <Navbar />
	    <TldCanvas />
	</>
  )
};

export default FreeDrawCanvas;
```

So by performing the above process we submit changes to the repo, got feedback, made changes and submitted

The above process can be repeated until everyone is satisfied with the quality of the code.

---
# Iterative Development Process Flowchart

It is approach where software is developed and refined through a series of iterations. Each iteration involves taking a small set of requirements, designing, coding, testing, and then integrating them into the existing codebase.

During iterative development we follow the above process

- We first create a new feature workflow

- We make the basic implementation for the required changes; where we make sure the we have the important parts of the code change correct (this prevents us from delivering something that was completely different from what was expected)

- We create a Pull request and get verification from the reviewer that the current implementation is in the same direction as is expected

- Once the feedback is received we make whatever changes are expected 

- We then submit another commit to the PR and wait for any more feedback; in the meanwhile we can work on improving code quality and add final touches to the feature being implemented

- If we receive any more feedback we implement those changes

- Finally we make PR that is supposed to be a final submission and is to be merged back into the development branch 

- This will lead us further into the `CI/CD` phase.

Below given is the iterative development process flowchart

_fun fact this diagram was made using the very changes described in this document_

<img width="1174" alt="iterative development flowchart" src="https://github.com/gnaaruag/wd401/assets/68043860/2113a64d-b4d3-453f-be54-9eb2cec8339e">

# Resolution of Merge conflicts

Merge conflicts occur when two or more developers make changes in the same area of the same file; this causes lack of clarity for version control which will then raise merge conflicts

Version control systems do the heavy lifting here and provide us with indicators as to where our conflicts occur

Conflicts will be indicated using markers like `>>>>>>>`, `<<<<<<<`, `=======` 

Lets consider a scenario where a merge conflict occurs with the `.tld-canvas` CSS class. We'll simulate two branches, `feature/update-tld-component` and `feature/update-tld-component-position`, making changes to the same CSS class.

in branch `feature/update-tld-component` we define `.tld-canvas` as follows

```css
.tld-canvas {
	position: absolute;
	top: 70px;
	right: 0;
	bottom: 0;
	left: 0;
}
```

in branch `feature/update-tld-component-position` we define `.tld-canvas` as follows

```css
.tld-canvas {
	position: fixed;
	top: 70px;
	right: 0;
	bottom: 0;
	left: 0;
}
```

On merging we get following conflict markers

```css
.tld-canvas {
<<<<<<< HEAD
	position: absolute;
=======
	position: fixed;
>>>>>>> feature/update-tld-component-position
	top: 70px;
	right: 0;
	bottom: 0;
	left: 0;
}
```

To resolve we must pick whichever segment is more relevant to our code; for example we pick the code from `feature/update-tld-component-position` our class looks as follows

```css
.tld-canvas {
	position: fixed;
	top: 70px;
	right: 0;
	bottom: 0;
	left: 0;
}
```

Once done we can commit these changes and push them for further process

```sh
git add tld-styles.css
git commit -m "Resolved merge conflict in .tld-canvas class"
```

In this scenario, a merge conflict occurred because both branches made conflicting changes to the `position` property of  `.tld-canvas` class. We resolved the conflict by editing the file to choose one of the `position` values. After resolving the conflict, we marked the file as resolved and committed the changes to complete the merge.

# CI/CD Integration

It is a good practice to use packages like `eslint`, `jest`, `prettier` to make sure code is correctly formatted and all tests pass

During CI/CD we run our code changes through adequate tests to cover all scenarios and merge our code into production only if all tests are passing, if any do not pass we make corrections to our code retest our test cases.

to carry out tests we use the react testing library and make tests like the following example

```tsx
test('renders TldCanvas component correctly', () => { 
	const { container } = render(<TldCanvas />); 
	const divElement = container.querySelector('div'); 
	expect(divElement).toBeInTheDocument();
	expect(divElement).toHaveStyle('position: fixed');
	expect(divElement).toHaveStyle('top: 70px'); 
});
```

We merge our code only when all test cases like these pass for our code

During CI/CD we can define pre commit hooks using husky to run test suites and lint our codebase; this helps massively in maintaining code quality and ensures good software development processes

we can  create a `.prettierrc` file to define linting style

```json
{ 
	"semi": true, 
	"singleQuote": true, 
	"trailingComma": "es5" 
	...
}
```

we add a `lint staged` attribute to  our `package.json` which will help us execute our pre-commit hooks with declarations made during husky setup

```json
{
	"lint-staged": { 
		"*.{js,jsx,ts,tsx}": [ "eslint --fix", 
			"prettier --write", 
			"jest --bail --findRelatedTests" 
		], "*.{json,css,scss,md}": [ 
			"prettier --write" 
		] 
	} 
}
```

the CI/CD for this application is currently done on Vercel which takes care of the building and deployment, but in the future levels when we are taught more about CI/CD i will define a custom CI/CD pipeline.

Therefore during this whole document we create a feature get it through code review, resolve any conflicts and then we see how our CI/CD handles it to deploy it into production
