

# Sails JS Documentation

## Actions
- Actions are the principal objects in your Sails application that are responsible for responding to requests from a web browser, mobile application or any other system capable of communicating with a server. They often act as a middleman between your models and views. With rare exceptions, the actions will orchestrate the bulk of your project’s business logic.

    ```
        async function (req, res) {
          return res.send('Hi there!');
    }
    ```
- Actions are defined in the api/controllers/ folder and subfolders
- it must be kebab-cased (containing only lowercase letters, numbers and dashes)
- When referring to an action in Sails, use its path relative to api/controllers, without any file extension. For example, the api/controllers/user/find.js file represents an action with the identity user/find.
- Use action2 format when writing actions. You can use ```sails generate action``` to quickly create an actions2 action.Or you can use ```classic actions```, but action2 format is recommended. You can use ```sails generate action with --no-actions2``` to quickly create a classic action.

## Controllers
- The quickest way to get started writing Sails apps is to organize your actions into controller files. A controller file is a PascalCased file whose name must end in Controller, containing a dictionary of actions. For example, a "User Controller" could be created at api/controllers/UserController.js file containing:
- You can use ```sails generate controller``` to quickly create a controller file.

    ```
        module.exports = {
          login: function (req, res) { ... },
          logout: function (req, res) { ... },
          signup: function (req, res) { ... },
        };
    ```

- For larger, more mature apps, standalone actions may be a better approach than controller files. In this scheme, rather than having multiple actions living in a single file, each action is in its own file in an appropriate subfolder of api/controllers.
    ```
        api/
         controllers/
          user/
           login.js
           logout.js
           signup.js
    ```
- In the tradition of most MVC frameworks, mature Sails apps usually have "thin" controllers—that is, your action code ends up lean because reusable code has been moved into helpers or occasionally even extracted into separate node modules. This approach can definitely make your app easier to maintain as it grows in complexity.
- Sails recommends this general rule of thumb: wait until you're about to use the same piece of code for the third time before you extrapolate it into a separate helper.


## Views
- In Sails, views are markup templates that are compiled on the server into HTML pages. In most cases, views are used as the response to an incoming HTTP request, e.g. to serve your home page.
- By default, Sails is configured to use EJS (Embedded Javascript) as its view engine.
- If you prefer to use a different view engine, there are a multitude of options
- If you don't need to serve dynamic HTML pages at all (say, if you're building an API for a mobile app), you can remove the directory from your app.

## Assets
- Assets refer to static files (js, css, images, etc.) on your server that you want to make accessible to the outside world
- When you lift your app, add files to your assets/ folder, or change existing assets, Sails' built-in asset pipeline processes and syncs those files to a hidden folder (.tmp/public/).
- The contents of this .tmp/public folder are what Sails actually serves at runtime. This is roughly equivalent to the "public" folder in express, or the www/ folder you might be familiar with from other web servers like Apache.
- Behind the scenes, Sails uses the serve-static middleware from Express to serve your assets. You can configure this middleware (e.g. to change cache settings) in ```/config/http.js.```


## Models and ORM
- Sails comes installed with a powerful ```ORM/ODM called Waterline```, a datastore-agnostic tool that dramatically simplifies interaction with one or more databases.
-  It provides an abstraction layer on top of the underlying database, allowing you to easily query and manipulate your data without writing vendor-specific integration code
- if your app uses a bunch of SQL queries, it will be very hard to switch to Mongo later, or Redis, and vice versa.
- **Waterline** query syntax floats above all that, focusing on business logic like creating new records, fetching/searching existing records, updating records, or destroying records. No matter what database you're contacting, the usage is exactly the same
- Since Sails uses **sails-disk** by default, you can start building your app with zero configuration, using a local temporary file as storage. When you're ready to switch to the real thing, just change your app's **datastore configuration**.
- Sails/Waterline lets you hook up different models to different datastores, locally or anywhere on the internet

## Datastores
- A datastore represents a particular database configuration.

```
    // in config/datastores.js
    // ...
    {
      adapter: 'sails-mysql',
      host: 'localhost',
      port: 3306,
      user: 'root',
      password: 'g3tInCr4zee&stUfF',
      database: 'database-name'
    }
```

## Models
- A model represents a set of structured data, called records. Models usually correspond to a table/collection in a database, attributes correspond to columns/fields, and records correspond to rows/documents.
```
    // api/models/Product.js
    module.exports = {
      attributes: {
        nameOnMenu: { type: 'string', required: true },
        price: { type: 'string', required: true },
        percentRealMeat: { type: 'number', defaultsTo: 20, columnType: 'FLOAT' },
        numCalories: { type: 'number' },
      },
    };
```
- Once a Sails app is running, its models may be accessed from within controller actions, helpers, tests, and just about anywhere else you normally write backend code.
- There are many built-in methods available on models, the most important of which are the model methods like .find() and .create(). More check [here!](https://sailsjs.com/documentation/reference/waterline-orm/models)
- Every model in Sails has a set of methods exposed on it to allow you to interact with the database in a normalized fashion. This is the primary way of interacting with your app's data.
- Since they usually have to send a query to the database and wait for a response, most model methods are **asynchronous**. Use async/await

## Configuration
- While Sails dutifully adheres to the philosophy of convention-over-configuration, it is important to understand how to customize those handy defaults from time to time
- Sails apps can be configured programmatically, by specifying environment variables or command-line arguments, by changing the local or global .sailsrc files, or (most commonly) using the boilerplate configuration files conventionally located in the config/ folder of new projects. The authoritative, merged-together configuration used in your app is available at runtime on the sails global as sails.config
- Settings specified in the standard configuration files will generally be available in all environments (i.e. development, production, test, etc.).
- Any files saved under the /config/env/<environment-name> folder will be loaded only when Sails is lifted in the <environment-name> environment. For example, files saved under config/env/production will only be loaded when Sails is lifted in production mode.

### The config/local.js file
- You may use the ```config/local.js``` file to configure a Sails app for your local environment (your laptop, for example). The settings in this file take precedence over all other config files except ```.sailsrc.```
- Since they're intended only for local use, they should not be put under version control (and are included in the default .gitignore file for that reason). Use local.js to store local database settings, change the port used when lifting an app on your computer, etc.

### Accessing sails.config in your app
- The config object is available on the Sails app instance (sails). By default, this is exposed on the global scope during lift, and therefore available from anywhere in your app.
    ```
        // This example checks that, if we are in production mode, csrf is enabled.
        // It throws an error and crashes the app otherwise.
        if (sails.config.environment === 'production' && !sails.config.security.csrf) {
          throw new Error('STOP IMMEDIATELY ! CSRF should always be enabled in a production deployment!');
        }
    ```
- read more [here!](https://sailsjs.com/documentation/concepts/configuration)

## Policies
- Policies in Sails are versatile tools for authorization and access control: they let you execute some logic before an action is run in order to determine whether or not to continue processing the request. The most common use-case for policies is to restrict certain actions to logged-in users only.
- NOTE: policies apply only to controllers and actions, not to views.
- It's best to avoid implementing numerous or complex policies in your app. Instead, when implementing features like granular, role-based permissions, rely on your actions to reject unwanted access.
- It is located in config/policies.js
- For more read [here!](https://sailsjs.com/documentation/concepts/policies)

## Helpers
-  Sails apps come with built-in support for helpers, simple utilities that let you share Node.js code in more than one place
- Helpers follow the same specification as shell scripts and actions2.
    ```
        // api/helpers/format-welcome-message.js
        module.exports = {

          friendlyName: 'Format welcome message',


          description: 'Return a personalized greeting based on the provided name.',


          inputs: {

            name: {
              type: 'string',
              example: 'Ami',
              description: 'The name of the person to greet.',
              required: true
            }

          },


          fn: async function (inputs, exits) {
            var result = `Hello, ${inputs.name}!`;
            return exits.success(result);
          }

        };
    ```
- The core of the helper is the fn function, which contains the actual code that the helper will run
- Note that, as opposed to a typical JavaScript function that uses return to provide output to the caller, helpers provide that result value by passing it in to exits.success().
- Exits describe all the different possible outcomes a helper can have, good or bad.
- By default, all helpers are considered asynchronous. While this is a safe default assumption, it's not always true. When you know for certain that your helper is synchronous, you can optimize performance by telling Sails using the sync: true property
- Generate a helper: ```sails generate helper foo-bar```
- Calling a helper: 
    ```var result = await sails.helpers.formatWelcomeMessage('Dolly');
    sails.log('Ok it worked!  The result is:', result);
    ```
- For more read [here!](https://sailsjs.com/documentation/concepts/helpers)

## Deployment: 
- [read here!](https://sailsjs.com/documentation/concepts/deployment)

## Blueprints
- Blueprints are Sails’s way of quickly generating API routes and actions based on your application design.
- Together, blueprint routes and blueprint actions constitute the blueprint API, the built-in logic that powers the RESTful JSON API you get every time you create a model and controller.
- Ex:
    ```
        For example, if you create a User.js model file in your project, then with blueprints enabled you will be able to immediately visit /user/create?name=joe to create a user, and visit /user to see an array of your app's users. All without writing a single line of code!
    ```
- Blueprints are a powerful tool for prototyping, but in many cases can be used in production as well, since they can be overridden, protected, extended or disabled entirely.

### Blueprint actions
- Blueprint actions are generic actions designed to work with your models
- Read more [here!](https://sailsjs.com/documentation/concepts/blueprints/blueprint-actions)

### Blueprint routes
- When you run sails lift with blueprints enabled, the framework inspects your models and configuration in order to bind certain routes automatically. 
- There are four types of blueprint routes in Sails:

#### RESTful blueprint routes
- REST blueprints are the automatically generated routes Sails uses to expose a conventional REST API for a model, including find, create, update, and destroy actions.

#### Shortcut blueprint routes
- Shortcut routes are a simple (development-mode only) hack that provides access to your models from your browser's URL bar.

#### Action shadow routes
- When action shadow routes (or "action shadows") are enabled, Sails will automatically create routes for your custom controller actions. This is sometimes useful (especially early on in the development process) for speeding up backend development by eliminating the need to manually bind routes. When enabled, GET, POST, PUT, and DELETE routes will be generated for every one of a controller's actions.



[Reference:!](https://sailsjs.com/documentation/concepts)






