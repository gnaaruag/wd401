### Scenario

You've joined a team where the integration and bundling of JavaScript (JS) is a crucial aspect of their modern web development stack. Let's explore a work-specific scenario based on the lessons you've covered in JS bundling.

You're assigned to a project that involves enhancing the user interface of an existing web application. Your task is to integrate JS seamlessly to improve the user experience. The team encourages the use of JS bundling tools like Webpack and is particularly interested in optimizing the bundling process.

### Problem Statement

In your new role as a Web Developer, you're assigned to a project that involves optimizing the integration of JavaScript into a web application. The team emphasizes the importance of efficient JS bundling for enhanced application performance. As you embark on the development task, challenges related to bundling, code splitting, lazy loading, and the implementation of import maps surface. Your role is to address these challenges, showcasing your ability to optimize JS integration and explore advanced bundling techniques. The team is particularly interested in your practical application of concepts such as code splitting, lazy loading, and import maps to improve the application's overall performance.

### Application context

Before I provide my solution to L1, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

---

### Webpack configuration

To better handle modules, assets, `css` and `js` modules in the project. we define a webpack config in the `webpack.config.js` file

```js
// webpack.config.js
import path from 'path';
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

export default {
  mode: "development", // Set mode to 'development'
  entry: "./src/main.tsx",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),\
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [{
            loader: 'ts-loader',
            options: {
              compilerOptions: {
                noEmit: false,
              },
            },
          },],
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        use: ["file-loader"],
        type: "asset/resource",
      },
    ],
  },
};
```

This enables us to better handle `css`, `jsx`, `tsx` in our project.

To ensure the webpack is compiled into the `dist` folder properly we make the following declarations in our application

```json
// package.json

...
"scripts": {
	...,
	"wp-start": "webpack serve --open",
    "wp-build": "npx webpack build",
    ...,
}

...
```

By running `wp-build` the webpack bundle gets compiled into the `dist` folder
By running  `wp-start` the webpack is served

### Improvements to this configuration

#### Lazy loading

As of now everything gets imported on call directly; but to improve code efficiency we can use lazy loading features

Usually, we import components with the static import declaration:

To defer loading this component’s code until it’s rendered for the first time, we replace this import with:

Lets see an example for this

```tsx
// freedraw.tsx
import React, { lazy } from 'react';

const TldCanvas = lazy(() => import('./TldCanvas.tsx'));

const Freedraw: React.FC = () => {
	<TldCanvas/>
}

export default Freedraw;
```

#### Dynamic importing

Lets also take a look at dynamic imports as a way of optimization

```tsx
import React from 'react'; 
const LazyButton: React.FC = () => { 
	const handleClick = async () => { 
		const module = await import('./lazyModule'); module.default(); 
	}; 
	
	return ( 
	<button id="lazy-button" onClick={handleClick}> 
		Load Lazy Module 
	</button> ); 
}; 

export default LazyButton;
```

Therefore the import only occurs when we need the component

this saves us resources during initial rendering as we only import components we need

Here's what we achieve by doing the above two 

- Reduction in loading time
- An improvement in performance
- Improved UX

We can also show loaders using `Suspense` while the component is being loaded into the `dom`

### Import maps

Import maps are a configuration file that specifies how JavaScript modules are resolved and loaded in a web application.

To achieve this we create a file called `importmap.json`, an example import map will look as follows

```json
// importmap.json
{
  "imports": {
    "@/components/CanvasComponent": "/src/components/CanvasComponent.tsx",
    "@/components/TldCanvas": "/src/components/TldCanvas.tsx",
    "@/pages/Freedraw": "/src/pages/Freedraw.tsx",
    "@/components/Navbar": "/src/components/Navbar.tsx"
  }
}
```

Lets see the below component to illustrate how this `importmap` definition would come into play

```tsx
import React from 'react'; 
import Navbar from '@/components/Navbar'; 
import TldCanvas from '@/components/TldCanvas'; 
const App: React.FC = () => { 
	return ( 
		<div> 
			<Navbar /> 
			<main> 
				<TldCanvas /> 
			</main>  
		</div> 
	); 
}; 

export default App;
```

This helps us with 
- Reducing import complexing
- Logically grouping import components
- We get to use components using aliases instead of typing the whole route every time

By doing the above we have integrated import maps and webpack into the application

