# aws-app-sync

Easily deploy AWS AppSync apps with [Serverless Components](https://github.com/serverless/components).

## Table of Contents

1. [Install](#1-install)
2. [Create](#2-create)
3. [Configure](#3-configure)
4. [Deploy](#4-deploy)

### 1. Install

```shell
$ npm install -g serverless
```

### 2. Create

Just create the following simple boilerplate:

```shell
$ touch serverless.yml # more info in the "Configure" section below
$ touch index.js       # your lambda code
$ touch .env           # your AWS api keys
```

```
# .env
AWS_ACCESS_KEY_ID=XXX
AWS_SECRET_ACCESS_KEY=XXX
```

the `index.js` file should look something like this:

```js
module.exports.createUser = async (e) => {
  return {
    statusCode: 200,
    body: 'Created User'
  }
}

module.exports.getUsers = async (e) => {
  return {
    statusCode: 200,
    body: 'Got Users'
  }
}

module.exports.auth = async (event, context) => {
  return {
    principalId: 'user',
    policyDocument: {
      Version: '2012-10-17',
      Statement: [
        {
          Action: 'execute-api:Invoke',
          Effect: 'Allow',
          Resource: event.methodArn
        }
      ]
    }
  }
}
```

Keep reading for info on how to set up the `serverless.yml` file.

### 3. Configure

You can configure the component to either create a new REST API from scratch, or extend an existing one.

#### Creating REST APIs

You can create new REST APIs by specifying the endpoints you'd like to create, and optionally passing a name and description for your new REST API. You may also choose between a lambda proxy or http proxy integration by using the function or proxyURI field respectively. The function field will override the proxyURI field.

```yml
# serverless.yml

createUser:
  component: '@serverless/aws-lambda'
  inputs:
    code: ./code
    handler: index.createUser
getUsers:
  component: '@serverless/aws-lambda'
  inputs:
    code: ./code
    handler: index.getUsers
auth:
  component: '@serverless/aws-lambda'
  inputs:
    code: ./code
    handler: index.auth

restApi:
  component: '@serverless/aws-api-gateway'
  inputs:
    description: Serverless REST API
    endpoints:
      - path: /users
        method: POST
        function: ${createUser.arn}
        authorizer: ${auth.arn}
      - path: /users
        method: GET
        function: ${getUsers.arn}
        authorizer: ${auth.arn}
      - path: /users
        method: PUT
        proxyURI: https://example.com/users
        authorizer: ${auth.arn}
```

#### Extending REST APIs

You can extend existing REST APIs by specifying the REST API ID. This will **only** create, remove & manage the specified endpoints without removing or disrupting other endpoints.

```yml
# serverless.yml

createUser:
  component: '@serverless/aws-lambda'
  inputs:
    code: ./code
    handler: index.createUser
getUsers:
  component: '@serverless/aws-lambda'
  inputs:
    code: ./code
    handler: index.getUsers

restApi:
  component: '@serverless/aws-api-gateway'
  inputs:
    id: qwertyuiop # specify the REST API ID you'd like to extend
    endpoints:
      - path: /users
        method: POST
        function: ${createUser.arn}
      - path: /users
        method: GET
        function: ${getUsers.arn}
```

### 4. Deploy

```shell
$ serverless
```

&nbsp;

### New to Components?

Checkout the [Serverless Components](https://github.com/serverless/components) repo for more information.
