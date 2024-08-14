Automating Node.js Application Deployment with Docker and CI/CD Pipelines
### Application context

Before I provide my solution to L7, I will explain the premise of the application.

The application Pairdraw is a doodle sharing application. In the application the user is supposed to form pairs with friends by sharing invite codes; once a `pair` is formed the users can share between themselves doodles.

In this milestone submission I will be demonstrating how to Dockerize a node js application and automate deployment using CI/CD pipelines.

To showcase this we will be using the backend sub repo of the Pairdraw application which is built in node.js 

### Dockerizing a Node.js app

We will first define a `.dockerfile`

our `.dockerfile` will look as shown below

```dockerfile
FROM --platform=$BUILDPLATFORM node:lts-alpine as base
WORKDIR /app
COPY package.json /
EXPOSE 3000

FROM base as development
ENV NODE_ENV=development
RUN npm install -g nodemon && npm install
COPY . /app
CMD npm run start

FROM base as production
ENV NODE_ENV=production
RUN npm install
COPY . /app
CMD node server.js
```

the above file is the same as one shown in tutorials with reasonable modifications made to adjust to the code. 

### Environment Variables

also here's how the `.env` file looks 

```env
PORT=3000
DB=mongodb+srv://user:password@pairdraw.randomtext.mongodb.net/?retryWrites=true&w=majority&appName=pairdraw
CLERK_SECRET_KEY="sk_test_THISISASECRETFORAUTH"
TEST_DB_URI=mongodb://localhost:27017/pairing_test
```

`PORT`: defines what server port is exposed
`DB`: defines a route to the remote mongo db atlas deployment
`CLERK_SECRET_KEY`: clerk is a third party auth provider, this key defines interactions with service for backend
`TEST_DB_URI`: defines route for testing database

We run the following commands to build and run our container

```bash
docker build -t back-img .  
docker run -d -p 3000:3000 --name back-img --env-file .env back-img  
```

we get the following on the docker desktop application
![[docker desktop.png]]

### Docker Compose

Now we also need to take care of dockerizing, the frontend and making sure that both run in harmony

Dockerfile for a React-Ts app is written as follows

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 5173

CMD ["nginx", "-g", "daemon off;"]
```

To build and run we use the following commands

```bash
docker build -t react-ts-app .
docker run -d -p 80:80 --name react-ts-container react-ts-app
```

we get the following on the docker desktop application
![[docker desktop 2.png]]

Now to make both docker containers run properly we write a `docker-compose.yml` file that looks like the following.

```yml
version: '3.8'

services:
  backend:
    build: ./pairdraw-back
    ports:
      - "3000:3000"
    env_file:
      - ./pairdraw-back/.env
    networks:
      - app-network

  frontend:
    build: ./pairdraw-front
    ports:
      - "80:80"
    depends_on:
      - backend
    environment:
      - REACT_APP_BACKEND_URL=http://backend:3000
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

to run it we use

```bash
docker-compose up -d
```

**Note: We don't need to configure our database since it is hosted in a mongoDB cloud instance**

### CI/CD

Here is a `main.yml` file defined in  `.github/workflows/main.yml` 

```yml
name: CI/CD Pipeline

on:
  push:
    branches:
      - master

jobs:
  
  build-and-test:
  
	runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

	- name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'

    - name: Install dependencies
      run: |
        cd pairdraw-back
        npm install

    - name: Run tests
      run: |
        cd pairdraw-back
        npm test
    env:
      TEST_DB_URI: ${{ secrets.TEST_DB_URI }}

  deploy:

    runs-on: ubuntu-latest
    needs: build-and-test
    if: needs.build-and-test.result == 'success'
    steps:

	  - name: Deploy to production
        env:
          deploy_url: ${{ secrets.RENDER_DEPLOY_HOOK }}
        run: |
          curl "$deploy_url"

  notify:

	needs: [build-and-test, deploy]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Send Slack notification on success
        if: needs.build-and-test.result == 'success' && needs.deploy.result == 'success'
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: |
            {
              "text": "CI/CD process succeeded!"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_BOT_TOKEN }}
      - name: Send Slack notification on failure

		if: needs.run-tests.result != 'success' || needs.deploy.result != 'success'
        uses: slackapi/slack-github-action@v1.25.0
        with:
          payload: |
            {
              "text": "*${{ github.workflow }}* failed. Access the details https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}."
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_BOT_TOKEN }}
```

Above `yml` file depends on following secrets defined in the GitHub repo secrets 
- `TEST_DB_URI`
- `RENDER_DEPLOY_HOOK`
- `SLACK_BOT_TOKEN`

Our CI/CD workflow consists of following steps
- Build and Test - via `npm run build` and `npm run test`
- Deploy - A webhook to the render web service is defined, app is deployed to render when tests pass
- Notify - A slack app checks for incoming web hooks which notifies CI pass or fail in defined slack channel

All the GitHub environments will be defined in `settings/secrets/actions`

on failure of a GitHub workflow we get the following text in a slack channel
![[slack failure.png]]

when the workflow succeeds we get the following text in designated slack channel 
![[slack success.png]]

Once our CI/CD pipeline successfully executes we will see the below in our workflow
![[successful ci pipeline.png]]
