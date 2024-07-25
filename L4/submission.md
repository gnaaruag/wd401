### Application context

Before I provide my solution to L4, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

### Configuring tests

We utilize Jest testing to run unit tests on our codebase. This will be configured in the `package.json`. To run the testing suite we run `npm run test` where we define the test in `package.json` as follows

```json
"scripts": {
	"dev": "nodemon index.js",
	"start": "node index.js",
	"test": "set NODE_ENV=test&&  jest --detectOpenHandles"
}
```

Given proper setup this executes all tests and checks validity

### Test coverage

It is a measure of how much of your codebase is exercised by your tests. It helps ensure that your tests are comprehensive and can identify untested parts of the code, which may contain bugs.

Key aspects are Line coverage, branch coverage, function coverage, path coverage.

Maximum coverage ensures better code reliability

Usually unit tests  cover individual components and functions, integration tests cover API endpoints, E2E tests cover interactions

A few libraries we use are  `Jest`, `Supertest`, `Cypress` etc.

### Test Suite execution via GitHub Actions

We can configure a workflow in `.github/workflows/main.yml` which runs on every push via GitHub actions. During this we setup a database connection, checkout the repo along with the changes, install dependencies and run tests in jest once all tests pass it will then make the push commit, else the push gets rejected.

Therefore by defining all of the above mentioned we can setup a robust and reliable testing setup and environment. Actions config ensures continuous testing is conducted and any error/but gets caught before evolving into a larger problem. This helps improve code quality and ensures reliability and correct functionality 
### Execution

Below given is demonstration of the execution for the milestone requirements

first we ensure to install all packages, our `package.json` should contain these

```json
 "devDependencies": {
    "cypress": "^13.6.4",
    "eslint": "^8.55.0",
    "husky": "^8.0.3",
    "jest": "^29.7.0",
    "lint-staged": "^15.2.0",
    "nodemon": "^3.0.2",
    "prettier": "^3.1.0",
    "mongoose": "^6.6.2",
    "mongo": "^6.1.5"
    "supertest": "^6.3.3"
  }
```

#### Jest tests

Below given is the test suite that was defined for our application
```js
//pairing.test.js
const request = require('supertest');
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const routes = require('../routes/pairing.route'); // Adjust the path if necessary
const Pairing = require('../models/pairing.model');

dotenv.config();

const app = express();

app.use(express.json());
app.use('/api', routes);

beforeAll(async () => {
  const uri = process.env.TEST_DB_URI; // Ensure you have a TEST_DB_URI in your .env file
  await mongoose.connect(uri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
});

beforeEach(async () => {
  await Pairing.deleteMany({});
});

afterAll(async () => {
  await mongoose.connection.close();
});
  

describe('Pairing API', () => {
  test('should create a new pair', async () => {
    const response = await request(app)
      .post('/api/new-pair')
      .send({
        pairName: 'Test Pair',
        adminUser: 'admin@test.com',
      });
    expect(response.status).toBe(200);
    expect(response.text).toBe('Object Created');
    const pairings = await Pairing.find({});
    expect(pairings.length).toBe(1);
    expect(pairings[0].pairName).toBe('Test Pair');
  });
  
  test('should get all user pairs', async () => {
    const pair1 = new Pairing({
      id: 'pair1',
      pairName: 'Pair 1',
      adminUser: 'admin1@test.com',
    });
    const pair2 = new Pairing({
      id: 'pair2',
      pairName: 'Pair 2',
      adminUser: 'admin2@test.com',
      otherUser: 'user@test.com',
    });
    await pair1.save();
    await pair2.save();
    const response = await request(app)
      .post('/api/get-all-pairs')
      .send({
        user: 'user@test.com',
      });
    expect(response.status).toBe(200);
    expect(response.body.length).toBe(1);
    expect(response.body[0].pairName).toBe('Pair 2');
  });

  test('should add user to pair', async () => {
    const pair = new Pairing({
      id: 'pair1',
      pairName: 'Pair 1',
      adminUser: 'admin1@test.com',
    });
    await pair.save();
    const response = await request(app)
      .post('/api/add-to-pair')
      .send({
        pairID: 'pair1',
        user: 'user@test.com',
      });
    expect(response.status).toBe(200);
    expect(response.body.otherUser).toBe('user@test.com');
    const updatedPair = await Pairing.findOne({ id: 'pair1' });
    expect(updatedPair.otherUser).toBe('user@test.com');
  });

  test('should upload an image', async () => {
    const pair = new Pairing({
      id: 'pair1',
      pairName: 'Pair 1',
      adminUser: 'admin1@test.com',
    });
    await pair.save();
    const response = await request(app)
      .post('/api/upload-image')
      .send({
        pairID: 'pair1',
        user: 'admin1@test.com',
        image: 'base64imagestring',
      });
    expect(response.status).toBe(200);
    expect(response.body.imageA).toBe('base64imagestring');
    const updatedPair = await Pairing.findOne({ id: 'pair1' });
    expect(updatedPair.imageA).toBe('base64imagestring');
  });

  test('should send image', async () => {
    const pair = new Pairing({
      id: 'pair1',
      pairName: 'Pair 1',
      adminUser: 'admin1@test.com',
      imageA: 'base64imagestringA',
      imageB: 'base64imagestringB',
    });
    await pair.save();
    const response = await request(app)
      .post('/api/get-image')
      .send({
        pairID: 'pair1',
        user: 'admin1@test.com',
      });
    expect(response.status).toBe(200);
    expect(response.body.data).toBe('base64imagestringB');
  });
});
```

Here's how the output of that looks like

![jest output](https://github.com/user-attachments/assets/882eac29-42a4-47a6-aef9-3ad1521c5b6f)

#### Cypress E2E

Now we define End to end tests via cypress
 
```js
// cypress/e2e/pairing.cy.js
describe('Pairing API', () => {
	const pairData = {
      pairName: 'Test Pair',
      adminUser: 'admin@test.com',
    };
    
    it('should create a new pair', () => {
      cy.request('POST', '/api/new-pair', pairData).then((response) => {
        expect(response.status).to.eq(200);
        expect(response.body).to.eq('Object Created');
      });
    });

    it('should get all user pairs', () => {
      cy.request('POST', '/api/new-pair', pairData).then(() => {
        cy.request('POST', '/api/get-all-pairs', { user: 'admin@test.com' }).then((response) => {
          expect(response.status).to.eq(200);
          expect(response.body.length).to.be.greaterThan(0);
          expect(response.body[0].pairName).to.eq('Test Pair');
        });
      });
    });

    it('should add user to pair', () => {
      cy.request('POST', '/api/new-pair', pairData).then((response) => {
        const pairID = response.body.id;
        cy.request('POST', '/api/add-to-pair', { pairID, user: 'user@test.com' }).then((response) => {
          expect(response.status).to.eq(200);
          expect(response.body.otherUser).to.eq('user@test.com');
        });
      });
    });

    it('should upload an image', () => {
      cy.request('POST', '/api/new-pair', pairData).then((response) => {
        const pairID = response.body.id;
        cy.request('POST', '/api/upload-image', {
          pairID,
          user: 'admin@test.com',
          image: 'base64imagestring',
        }).then((response) => {
          expect(response.status).to.eq(200);
          expect(response.body.imageA).to.eq('base64imagestring');
        });
      });
    });

    it('should send image', () => {
      cy.request('POST', '/api/new-pair', pairData).then((response) => {
        const pairID = response.body.id;
        cy.request('POST', '/api/upload-image', {
          pairID,
          user: 'admin@test.com',
          image: 'base64imagestringA',
        }).then(() => {
          cy.request('POST', '/api/upload-image', {
            pairID,
            user: 'user@test.com',
            image: 'base64imagestringB',
          }).then(() => {
            cy.request('POST', '/api/get-image', {
              pairID,
              user: 'admin@test.com',
            }).then((response) => {
              expect(response.status).to.eq(200);
              expect(response.body.data).to.eq('base64imagestringB');
            });
          });
        });
      });
    });
  });
```
![cypress output](https://github.com/user-attachments/assets/77bc6e26-4e9c-4332-b36e-b5692b0d8074)

Here's the output of running the Cypress suite


#### GitHub Actions

We define this workflow in `.github/workflow/main.yml`

```yml
name: CI Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:latest
        ports:
          - 27017:27017
        options: >-
          --health-cmd "mongo --eval 'db.runCommand({ ping: 1 })'"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Navigate to backend
        run: cd pairdraw-back && echo "Now in $(pwd)"
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - name: Install dependencies
        run: npm install
      - name: Run Jest tests
        env:
          TEST_DB_URI: mongodb://mongodb:27017/testdb
        run: npm test
      - name: Run Cypress tests
        run: npx cypress run
      - name: Archive Cypress test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: cypress-results
          path: |
            cypress/screenshots
            cypress/videos
            cypress/reports
```

This workflow runs on every push commit and runs tests after commit
