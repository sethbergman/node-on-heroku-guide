## Node.js on Heroku Guide
[Heroku Dev Center](https://devcenter.heroku.com/articles/getting-started-with-nodejs)

A barebones Node.js app using [Express 4](http://expressjs.com/).

This application supports the [Getting Started with Node on Heroku](https://devcenter.heroku.com/articles/getting-started-with-nodejs) article - check it out.

## Running Locally

### Prepare the app

```bash
git clone https://github.com/heroku/node-js-getting-started.git
cd node-js-getting-started
```
### Setup Remote Repository
```bash
git remote -v
origin  https://github.com/heroku/node-js-getting-started.git (fetch)
origin  https://github.com/heroku/node-js-getting-started.git (push)

git remote set-url --push origin https://github.com/sethbergman/node-on-heroku-guide.git

git remote -v
origin  https://github.com/heroku/node-js-getting-started.git (fetch)
origin  https://github.com/sethbergman/node-on-heroku-guide.git (push)
```
### Deploy the app
```bash
heroku create
```
Now deploy your code:
```bash
git push heroku master
```
The application is now deployed. Ensure that at least one instance of the app is running:
```bash
heroku ps:scale web=1
```
Now visit the app at the URL generated by its app name. As a handy shortcut, you can open the website as follows:
```bash
heroku open
```

### View logs
Heroku treats logs as streams of time-ordered events aggregated from the output streams of all your app and Heroku components, providing a single channel for all of the events.

View information about your running app using one of the [logging commands](https://devcenter.heroku.com/articles/logging), `heroku logs`:
```bash
heroku logs --tail
```
Visit your application in the browser again, and you’ll see another log message generated.
Press `Control+C` to stop streaming the logs.

### Define a Procfile
Use a [Procfile](https://devcenter.heroku.com/articles/procfile), a text file in the root directory of your application, to explicitly declare what command should be executed to start your app.

The `Procfile` in the example app you deployed looks like this:
```bash
web: node index.js
```
This declares a single process type, `web`, and the command needed to run it. The name `web` is important here. It declares that this process type will be attached to the [HTTP routing stack](https://devcenter.heroku.com/articles/http-routing) of Heroku, and receive web traffic when deployed.

Procfiles can contain additional process types. For example, you might declare one for a background worker process that processes items off of a queue.

### Scale the app
Right now, your app is running on a single web [dyno](https://devcenter.heroku.com/articles/dynos). Think of a dyno as a lightweight container that runs the command specified in the `Procfile`.

You can check how many dynos are running using the `ps` command:
```bash
heroku ps
```

### Declare app dependencies
Heroku recognizes an app as Node.js by the existence of a `package.json` file in the root directory. For your own apps, you can create one by running `npm init`.

The demo app you deployed already has a `package.json`, and it looks something like this:
```
{
  "name": "node-js-getting-started",
  "version": "0.1.4",
  ...
  "dependencies": {
    "ejs": "^2.3.1",
    "express": "~4.9.x"
  },
  ...
  "engines": {
    "node": "0.12.2"
  },
}
```
The `package.json` file determines both the version of Node.js that will be used to run your application on Heroku, as well as the dependencies that should be installed with your application.

When an app is deployed, Heroku reads this file and installs the appropriate node version together with the dependencies using the `npm install` command.

Run this command in your local directory to install the dependencies, preparing your system for running the app locally:
```bash
npm install
```
Due to a bug in npm, you may receive an error message on Windows that reads something like Error: ENOENT, stat `C:\Users\YOURUSERNAME\AppData\Roaming\npm`. If you do, manually create this directory with `mkdir C:\Users\YOURUSERNAME\AppData\Roaming\npm` and rerun `npm install`.

Once dependencies are installed, you will be ready to run your app locally.

### Run the app locally
Now start your application locally using Foreman, which was installed as part of the Toolbelt:
```bash
foreman start web
```
Just like Heroku, Foreman examines the `Procfile` to determine what to run.
Your app will now be running at [localhost:5000](http://localhost:5000/). Test that it’s working with `curl` or a web browser, then Ctrl-C to exit.

Foreman doesn’t just run your app - it also sets “config vars”, something you’ll encounter in a later tutorial.

### Push local changes
In this step you’ll learn how to propagate a local change to the application through to Heroku. As an example, you’ll modify the application to add an additional dependency and the code to use it.
Modify `package.json` to include a dependency for `cool-ascii-faces`:

```
  "dependencies": {
    "ejs": "^2.3.1",
    "express": "~4.9.x",
    "cool-ascii-faces": "~1.3.x"
  },
```
Modify `index.js` so that it `requires` this module at the start. Also add a new route (`/cool`) that uses it. Your final code should look like this:
```
var cool = require('cool-ascii-faces');
var express = require('express');
var app = express();

app.set('port', (process.env.PORT || 5000));

app.use(express.static(__dirname + '/public'));

// views is directory for all template files
app.set('views', __dirname + '/views');
app.set('view engine', 'ejs');

app.get('/', function(request, response) {
  response.render('pages/index')
});

app.get('/cool', function(request, response) {
  response.send(cool());
});

app.listen(app.get('port'), function() {
  console.log('Node app is running on port', app.get('port'));
});
```
Now test locally:
```bash
npm install
foreman start
```
Visiting your application at [http://localhost:5000/cool](http://localhost:5000/cool)(, you should see cute faces displayed on each refresh: `( ⚆ _ ⚆ )`.

Now deploy. Almost every deploy to Heroku follows this same pattern. First, add the modified files to the local git repository:
```bash
git add .
```
Now commit the changes to the repository:
```bash
git commit -m "My cool new Node.js app running on Heroku!"
```
Now deploy, just as you did previously:
```bash
git push heroku master
```
Finally, check that everything is working:
```bash
heroku open
```
Navigate to the `/cool` route, and you should see another face.

### Provision add-ons
Add-ons are third-party cloud services that provide out-of-the-box additional services for your application, from persistence through logging to monitoring and more.

By default, Heroku stores 1500 lines of logs from your application. However, it makes the full log stream available as a service - and several add-on providers have written logging services that provide things such as log persistence, search, and email and SMS alerts when certain conditions are met.

Provision the papertrail logging add-on:
```bash
heroku addons:create papertrail
```
The add-on is now deployed and configured for your application. You can list add-ons for your app like so:
```bash
heroku addons
```
To see this particular add-on in action, visit your application’s Heroku URL a few times. Each visit will generate more log messages, which should now get routed to the papertrail add-on. Visit the papertrail console to see the log messages:
```bash
heroku addons:open papertrail
```
A console will open up, showing the latest log events, and providing you with an interface to search and set up alerts.

### Start a console
You can run a command, typically scripts and applications that are part of your app, in a [one-off dyno](https://devcenter.heroku.com/articles/one-off-dynos) using the `heroku run` command. It can also be used to launch a REPL process attached to your local terminal for experimenting in your app’s environment:
```bash
$ heroku run node
Running `node` attached to terminal... up, run.2132
Detected 512 MB available memory, 512 MB limit per process (WEB_MEMORY)
Recommending WEB_CONCURRENCY=1
>
```
If you receive an error, `Error connecting to process`, then you may need to [configure your firewall](https://devcenter.heroku.com/articles/one-off-dynos#timeout-awaiting-process).

When the console starts, it has nothing loaded other than the Node.js standard library. From here you can `require` some of your application files.

For example, you will be be able to run the following:
```
> var cool = require('cool-ascii-faces')
> cool()
( ⚆ _ ⚆ )
```
To get a real feel for how dynos work, you can create another one-off dyno and run the `bash` command, which opens up a shell on that dyno. You can then execute commands there. Each dyno has its own ephemeral filespace, populated with your app and its dependencies - once the command completes (in this case, `bash`), the dyno is removed.
```bash
$ heroku run bash
Running `bash` attached to terminal... up, run.3052
~ $ ls
Procfile  README.md  composer.json  composer.lock  vendor  views  web
~ $ exit
exit
```
Don’t forget to type `exit` to exit the shell and terminate the dyno.

### Define config vars
Heroku lets you externalise configuration - storing data such as encryption keys or external resource addresses in [config vars](https://devcenter.heroku.com/articles/config-vars).

At runtime, config vars are exposed as environment variables to the application. For example, modify `index.js` so that the method repeats an action depending on the value of the `TIMES` environment variable:
```
app.get('/', function(request, response) {
  var result = ''
  var times = process.env.TIMES || 5
  for (i=0; i < times; i++)
    result += cool();
  response.send(result);
});
```
Foreman will automatically set up the environment based on the contents of the `.env` file in your local directory. Create a `.env` file that has the following contents:
```bash
TIMES=2
```
If you run the app with `foreman start`, you’ll see two faces will be generated every time.
To set the config var on Heroku, execute the following:
```bash
heroku config:set TIMES=2
```
View the config vars that are set using `heroku config`:
```bash
heroku config
```
Deploy your changed application to Heroku to see this in action.

### Provision a database
The [add-on marketplace](https://elements.heroku.com/addons#data-stores) has a large number of data stores, from Redis and MongoDB providers, to Postgres and MySQL. In this step you will add a free Heroku Postgres Starter Tier dev database to your app.

Add the database:
```bash
heroku addons:create heroku-postgresql:hobby-dev
```
This creates a database, and sets a `DATABASE_URL` environment variable (you can check by running `heroku config`).

Edit your `package.json` file to add the [pg npm module](https://npmjs.org/package/pg) to your dependencies:
```
"dependencies": {
    "pg": "4.x",
    "ejs": "^2.3.1",
    "express": "~4.9.x",
    "cool-ascii-faces": "~1.3.x"
}
```
Type `npm install` to install the new module for running your app locally. Now edit your `index.js` file to use this module to connect to the database specified in your DATABASE_URL environment variable:
```
var pg = require('pg');

app.get('/db', function (request, response) {
  pg.connect(process.env.DATABASE_URL, function(err, client, done) {
    client.query('SELECT * FROM test_table', function(err, result) {
      done();
      if (err)
       { console.error(err); response.send("Error " + err); }
      else
       { response.render('pages/db', {results: result.rows} ); }
    });
  });
})
```
This ensures that when you access your app using the `/db` route, it will return all rows in the `test_table` table.

Deploy this to Heroku. If you access `/db` you will receive an error as there is no table in the database. Assuming that you have [Postgres installed locally](https://devcenter.heroku.com/articles/heroku-postgresql#local-setup), use the `heroku pg:psql` command to connect to the remote database, create a table and insert a row:
```bash
$ heroku pg:psql
psql (9.3.2, server 9.3.3)
SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
Type "help" for help.
=> create table test_table (id integer, name text);
CREATE TABLE
=> insert into test_table values (1, 'hello database');
INSERT 0 1
=> \q
```
Now when you access your app’s `/db` route, you will see something like this:

                                    Database Results
                                  * 1 - hello database
  
Read more about [Heroku PostgreSQL](https://devcenter.heroku.com/articles/heroku-postgresql).
A similar technique can be used to install [MongoDB or Redis add-ons](https://addons.heroku.com/#data-stores).

### Next steps
You now know how to deploy an app, change its configuration, view logs, scale, and attach add-ons.

Here’s some recommended reading. The first, an article, will give you a firmer understanding of the basics. The second is a pointer to the main Node.js category here on Dev Center:

*Read [How Heroku Works](https://devcenter.heroku.com/articles/how-heroku-works) for a technical overview of the concepts you’ll encounter while writing, configuring, deploying and running applications.

*Read [Deploying Node.js Apps on Heroku](https://devcenter.heroku.com/articles/deploying-nodejs) to understand how to take an existing Node.js app and deploy it to Heroku.

*Visit the [Node.js category](https://devcenter.heroku.com/categories/nodejs) to learn more about developing and deploying Node.js applications.


## Documentation

For more information about using Node.js on Heroku, see these Dev Center articles:

- [Getting Started with Node.js on Heroku](https://devcenter.heroku.com/articles/getting-started-with-nodejs)
- [Heroku Node.js Support](https://devcenter.heroku.com/articles/nodejs-support)
- [Node.js on Heroku](https://devcenter.heroku.com/categories/nodejs)
- [Best Practices for Node.js Development](https://devcenter.heroku.com/articles/node-best-practices)
- [Using WebSockets on Heroku with Node.js](https://devcenter.heroku.com/articles/node-websockets)
