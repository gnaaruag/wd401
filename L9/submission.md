### Adding Error Tracking to a React Project

### Application context

Before I provide my solution to L8, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

In this milestone submission I will be demonstrating how to add error tracking to an application
### Configure the React SDK for Sentry and setup

#### Install sentry

to install Sentry into a react project as a dependency we do 

```bash
npm install @sentry/react @sentry/tracing
```
#### Initialize Sentry for the project

the `src/main.tsx` file should look like this

```jsx
// src/main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import * as Sentry from '@sentry/react';
// import { BrowserTracing } from '@sentry/tracing';

import './index.css'
import { ClerkProvider } from '@clerk/clerk-react'

Sentry.init({
  dsn: "https://33a62e0bc5803b0b090e3100716d09b4@o4507855881502720.ingest.de.sentry.io/4507855888777296",
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration(),
  ],
  tracesSampleRate: 1.0,
  tracePropagationTargets: ["localhost", "https://pairdraw.vercel.app"],
  replaysSessionSampleRate: 0.1,
});

// Import your publishable key
const PUBLISHABLE_KEY = import.meta.env.VITE_CLERK_PUBLISHABLE_KEY
if (!PUBLISHABLE_KEY) {
  throw new Error("Missing Publishable Key")
}
ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ClerkProvider
    publishableKey={PUBLISHABLE_KEY}
      afterSignUpUrl="/home"
      afterSignInUrl="/home"
>
      <App />
    </ClerkProvider>
  </React.StrictMode>,
)
```

#### Uploading source maps

Sentry also lets us upload source maps to enable readable stack traces for Errors

to setup we run the command in the terminal

```bash
npx @sentry/wizard@latest -i sourcemaps
```

The cli asks us to make certain choices during installation, see below for details about setup

```
┌   Sentry Source Maps Upload Configuration Wizard 
│
◇   ──────────────────────────────────────────────────────────────────────────────────╮
│                                                                                     │
│  This wizard will help you upload source maps to Sentry as part of your build.      │
│  Thank you for using Sentry :)                                                      │
│                                                                                     │
│  (This setup wizard sends telemetry data and crash reports to Sentry.               │
│  You can turn this off by running the wizard with the '--disable-telemetry' flag.)  │
│                                                                                     │
│  Version: 3.26.0                                                                    │
│                                                                                     │
├─────────────────────────────────────────────────────────────────────────────────────╯
(node:17968) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)
│
▲  You have uncommitted or untracked files in your repo:
│
│  - pairdraw-front/package-lock.json
│
│  The wizard will create and update files.
│
◇  Do you want to continue anyway?
│  Yes
│
◆  Are you using Sentry SaaS or self-hosted Sentry?
│  ● Sentry SaaS (sentry.io)
│  ○ Sentry SaaS (sentry.io)
│  ● Sentry SaaS (sentry.io)
│  ○ Sentry SaaS (sentry.io)
◇  Are you using Sentry SaaS or self-hosted Sentry?
│  Self-hosted/on-premise/single-tenant
│
◇  Please enter the URL of your self-hosted Sentry instance.
│  https://sentry.io/
│
■  Please enter a valid URL. (It should look something like "https://sentry.mydomain.com/", got undefined)
│
◇  Please enter the URL of your self-hosted Sentry instance.
│  https://sentry.io/
│
◇  Do you already have a Sentry account?
│  Yes
│
●  If the browser window didn't open automatically, please open the following link to log into Sentry:
│  
│  https://sentry.io/account/settings/wizard/nhmyw3u7rgwyj5nd52rli4s2nle0d2woi1flvgh41pz14xb6m9e2kp3b0m2a9gv9/
│
◇  Login complete.
│
◇  Select your Sentry project.
│  pairdraw/javascript-react
│
◇  Which framework, bundler or build tool are you using?
│  tsc
│
◇  Installed @sentry/cli with NPM.
│
◇  Where are your build artifacts located?
│  .\out
│
◇  We couldn't find artifacts at .\out. Are you sure that this is the location that contains your build artifacts?
│  Yes, I am sure!
│
◆  Enabled source maps generation in tsconfig.json.
│
●  We recommend checking the modified file after the wizard finished to ensure it works with your build setup.
│
●  Added a sentry:sourcemaps script to your package.json.
│
◇  Do you want to automatically run the sentry:sourcemaps script after each production build?
│  Yes
│
◇  Is npm run build your production build command?
│  Yes
│
●  Added sentry:sourcemaps script to your build command.
│
◆  Added auth token to .sentryclirc for you to test uploading source maps locally.
│
◆  Created .sentryclirc.
│
◆  Added .sentryclirc to .gitignore.
│
◇  Are you using a CI/CD tool to build and deploy your application?
│  Yes
│
◇  Add the Sentry authentication token as an environment variable to your CI setup:

SENTRY_AUTH_TOKEN=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

│
▲  DO NOT commit this auth token to your repository!
│
◇  Did you configure CI as shown above?
│  I'll do it later...
│
●  Don't forget! :)
│
└  That's it - everything is set up!

   Test and validate your setup locally with the following Steps:

   1. Build your application in production mode.
      → For example, run npm run build.
      → You should see source map upload logs in your console.
   2. Run your application and throw a test error.
      → The error should appear in Sentry:
      → https://pairdraw.sentry.io/issues/?project=4507855888777296
   3. Open the error in Sentry and verify that it's source-mapped.
      → The stack trace should show your original source code.

   If you encounter any issues, please refer to the Troubleshooting Guide:
   https://docs.sentry.io/platforms/javascript/sourcemaps/troubleshooting_js

   If the guide doesn't help or you encounter a bug, please let us know:
   https://github.com/getsentry/sentry-javascript/issues
```

#### Sample Error

We can either simulate errors in our application or run sample errors on sentry to get a feel for the site

![[Pasted image 20240828220856.png]]

### Debugging Errors

We can see that the errors get logged to the sentry dashboard, we can carry out management and fixing operations.

To debug: Sentry gives detailed error reports in the dashboard. Once an error is captured, they will be displayed to the Sentry dashboard to view error details, including stack traces, error messages, user actions leading to the error, IP address, OS, browser, and more. This information aids in diagnosing and resolving issues efficiently.

This often times is sufficient information for debugging but browsers provide us with a debugger tool we can use to debug JavaScript

for example consider the below script

```js
let x = 10;
let y = 20;
let z = x + y;
debugger;

let amount = z * y;
console.log(amount)
```

We get various tools and information to help us effectively debug our code, see example below
![[Pasted image 20240828221733.png]]

As we can see from above tools like sentry and the js debugger can help us debug and handle errors in our code, ensuring good code quality and system reliability

