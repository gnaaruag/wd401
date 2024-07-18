### Scenario

You've been assigned to a project that involves developing a feature-rich web application. The team acknowledges the benefits of using compile-to-JS languages and believes it's essential to make an informed choice between TypeScript and Babel based on the project's requirements.

### Problem Statement

In your role as a developer, you're tasked with developing a feature-rich web application. The team recognizes the benefits of using compile-to-JS languages and is eager to make an informed choice between TypeScript and Babel. The project's success depends on your ability to navigate the specific challenges related to the strengths and weaknesses of each language. The team expects you to provide a comprehensive rationale for the chosen compile-to-JS language, considering factors such as the type system, advantages, and project-specific requirements.

### Application context

Before I provide my solution to L3, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

_**Note:** the frontend for this application is already in expected `typescript` format so we will be migrate the backend `api` endpoints according to the requirements_

### Comparative analysis

Typescript and Babel are both very useful tools, but they have different use cases; they are as follows

**Typescript:** Typescript is a scaffolding built on top of JavaScript in order to create a type system which helps developers create a type safe application

Typescript is a compile to JavaScript language where all the typescript code written gets converted back to JavaScript 

The main advantage of Typescript is that it lets us write JavaScript code that is statically typed

for example consider this `util` function from the Pairdraw application backend

`in plain JavaScript`
```js
// paircode-generator.js
function generateRandomBase64String(length) {
    const charset = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+=";
    let result = "";
    for (let i = 0; i < length; i++) {
        const randomIndex = Math.floor(Math.random() * charset.length);
        result += charset.charAt(randomIndex);
    }
    return result;
}

module.exports = generateRandomBase64String
```

`when converted to Typescript`

```ts
// paircode-generator.ts
function generateRandomBase64String(length: number): string {
    const charset = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+=";
    let result = "";
    for (let i = 0; i < length; i++) {
        const randomIndex = Math.floor(Math.random() * charset.length);
        result += charset.charAt(randomIndex);
    }
    return result;
}

export default generateRandomBase64String;
```

Few advantages of Typescript

- Early error detection
- Robust type safety
- Improved code quality
- Better code maintainability

**Babel:** Babel plays a vital role in web development by allowing code written using the latest ECMAScript features while ensuring compatibility with older browsers and environments.

Babel primarily is a JavaScript compiler that allows developers to use the latest ECMAScript features by transpiling them into older versions of JavaScript that are compatible with all browsers. While Babel itself doesn't have a built-in type system, it can be combined with tools like TypeScript for static typing.

Advantages:

- **Latest JavaScript**: Babel enables developers to write code using the latest JavaScript syntax all while ensuring compatibility across different environments.
- **Plugins:** Babel has a vast ecosystem of plugins that extend its capabilities, allowing developers to customize the compilation process. 
- **Flexibility:** Babel can be configured to meet specific project requirements, making it suitable for a wide range of use cases.
- **Integration:** Babel seamlessly integrates with existing build pipelines and other tools commonly used in JavaScript development.
### Project conversion

Converting a JavaScript project to TypeScript can be a difficult but useful thing to do, as it leads to improved code quality, better maintainability, and enhanced productivity. Let me share an example of converting a simple JavaScript project to TypeScript, highlighting the addition of type annotations and how it contributes to code quality.

Lets see an example for converting a project from `js` to `ts`

```js
// original controller
const newPairing = async (request, response) => {
  let idInstance = "";
  const thumbnailId = thumbnailGenerate();
  while (true) {
    idInstance = pairingID(8);
    const data = await Pairing.findOne({ id: idInstance });
    if (!data) {
      break;
    }
  }
  try {
    await Pairing.create({
      id: idInstance,
      pairName: request.body.pairName,
      adminUser: request.body.adminUser,
      thumbnail: thumbnailId,
    });
    response.status(200).send("Object Created");
  } catch (e) {
    response.send(e);
  }
};
```

Converted typescript code

```ts
// controller in typescript
import { Request, Response } from 'express';
import { Pairing } from './models';
import { pairingID, thumbnailGenerate } from './utils'; 

const newPairing = async (request: Request, response: Response): Promise<void> => {
  let idInstance: string = "";
  const thumbnailId: string = thumbnailGenerate();
  while (true) {
    idInstance = pairingID(8);
    const data = await Pairing.findOne({ id: idInstance });
    if (!data) {
      break;
    }
  }
  try {
    await Pairing.create({
      id: idInstance,
      pairName: request.body.pairName,
      adminUser: request.body.adminUser,
      thumbnail: thumbnailId,
    });
    response.status(200).send("Object Created");
  } catch (e) {
    response.status(500).send(e);
  }
};

export default newPairing;
```

### Babel Config

Configuring Babel to transpile ES6+ code to ES5 involves setting up presets and plugins in a `.babelrc` file or using Babel configuration directly in `package.json`. To illustrate here's a typical Babel configuration 

```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "browsers": ["last 2 versions", "ie >= 11"]
      },
      "useBuiltIns": "usage",
      "corejs": "3.0.0"
    }]
  ],
  "plugins": []
}
```

**Explanation:** 
- @babel/preset-env preset is used to automatically determine the plugins and polyfills needed based on the specified environment targets. 
- targets: Specify the browsers or environments to support
- corejs: Specifies the version of core-js to use

### Case Study Presentation

Pairdraw is a multiuser application that has has more than 50 users in the past month. The system includes a React frontend application, a backend server. These two applications are decoupled, the frontend server is dependent on the backend server which in turn communicates with the database to display complete user info.

Below are few factors to consider about the project which may effect the migration to `compile to js` 

1. Project Size
	- The application is expected to scale and serve a large amount of people if it needs to
	- Therefore maintainability and scalability are important factors to consider
2. Tooling and ecosystem
	- Assess the compilers, linters, build systems available, compare and pick what's best for the project
	- Evaluate if the ecosystem and community support for libraries and frameworks the project relies on.
3. Performance overhead
	- We need to consider if there is any noticeable performance overhead which can be critical to the application
	- In our case the application doesn't deal with any time sensitive aspects so that is a tradeoff we can make to include stronger type safety
4. Future Maintainability
- The system is a long-term project expected to evolve over time with new features and updates.
- We need a language that promotes future maintainability, allowing for easy debugging, refactoring, and extension of the codebase.

##### Advantages of using typescript for Pairdraw

- Static typing provides enhanced code quality and error detection, particularly beneficial for large-scale projects like the CRM system.
- TypeScript is a superset of JavaScript, allowing gradual adoption and easy integration with existing JavaScript code.
- Strong community support and extensive tooling (e.g., TypeScript compiler, IDE support) facilitate development and maintenance.

##### Disadvantages of using typescript for Pairdraw

- Requires additional step to compile and the overhead can cost in terms build time etc.
- Initial setup and configuration might take more time compared to vanilla JavaScript or other compile-to-JavaScript languages.

#### Advantages of using Babel for Pairdraw

- Babel allows developers to use the latest JavaScript features and syntax (e.g., ES6+) while ensuring backward compatibility with older browsers.
- No additional learning curve for developers already proficient in JavaScript.
- Widely used in the JavaScript ecosystem with extensive plugin ecosystem for customization.

#### Disadvantages of using Babel for Pairdraw

- Lack of static typing may lead to more runtime errors, especially in larger projects.
- Configuration and management of Babel plugins can be complex, requiring careful consideration of compatibility and performance.

### Takeaways from above given analysis

**The decision to migrate to typescript has been made**

- Added type safety to the comes at little to no cost in terms of overhead
- The application isn't large enough to worry about compile time costs in the CI
- The backend is pretty easily migrate able to typescript 
- Comes with a better developer experience and stronger ecosystem support as Typescript has been an industry standard for half a decade at least

### Advanced Typescript features

##### Advanced TypeScript Features: Decorators and Generics

To demonstrate usage of Decorators and generics refer the below code 

For our intents and purposes we will use decorators for way to add metadata and modify behavior of classes, methods, or properties., while generics will be applied to make reusable components

consider for example the below decorator definitions

```ts
// decorators.ts
export function Controller(basePath: string): ClassDecorator {
    return (target) => {
        Reflect.defineMetadata('basePath', basePath, target);
    };
}
```

```ts
// decorators.ts
export function Route(method: string, path: string): MethodDecorator {
    return (target, propertyKey) => {
        if (!Reflect.hasMetadata('routes', target.constructor)) {
            Reflect.defineMetadata('routes', [], target.constructor);
        }
        const routes = Reflect.getMetadata('routes', target.constructor);
        routes.push({
            method,
            path,
            handler: propertyKey
        });
        Reflect.defineMetadata('routes', routes, target.constructor);
    };
}
```

consider the below generics definition

```ts
// types.ts
export interface Request<T = any> {
    body: T;
}

export interface Response<T = any> {
    status: (code: number) => Response;
    send: (body: T) => Response;
}
```

Here's a sample controller we write using the aforementiond definitions

```ts
// pairing.controller.ts

@Controller('/pairing')
class PairingController {
	@Route('post', '/new')
	@Route('post', '/new') async newPairing(
		request: Request<{ pairName: string; adminUser: string }>, 
		response: Response<string>) { 
		let idInstance = ''; 
		const thumbnailId = thumbnailGenerate(); 
		while (true) { 
			idInstance = pairingID(8); 
			const data = await Pairing.findOne({ id: idInstance }); 
			if (!data) { 
				break; 
				} 
		} 
		try { 
			await Pairing.create({ 
			id: idInstance, 
			pairName: request.body.pairName, 
			adminUser: request.body.adminUser, 
			thumbnail: thumbnailId, 
			}); 
			response.status(200).send('Object Created'); 
		} catch (e) { 
			response.send(e.message); 
		} 
	}
}
```


Considering above all factors and cases we have started migrating our codebase to typescript.
