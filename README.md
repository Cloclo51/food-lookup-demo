# Bug
**NOTE:** This fork isn't doing anything interesting; it is only being used as a reference in a bug.


## "Simple" repro steps
To repro my issue:

1. Clone this repository:

    `$ git clone https://github.com/vrk/food-lookup-demo.git`

2. Attempt to deploy to Heroku:

   ```
   $ cd food-lookup-demo
   $ heroku create
   $ git push heroku master
   ```

3. Enable the "api" dyno
   
   a. Navigate to https://dashboard.heroku.com
   b. Click on your newly created app -- see that in the "Dyno formation" table, "api" (the server process defined in the Procfile) is turned off
   c. Click "Configure dynos"
   d. In the "api" row, click the pencil icon on the right
   e. Toggle the "api" dyno on and press "Confirm"

4. Verify the "api" dyno is now an active process by going back to the terminal and running the following command:

   `heroku ps`

    which shows:
   
   ```
   Free dyno hours quota remaining this month: 938h 12m (93%)
   For more information on dyno sleeping and how to upgrade, see:
   https://devcenter.heroku.com/articles/dyno-sleeping
   
   === web (Free): cd client && npm start (1)
   web.1: up 2016/09/13 11:52:26 -0700 (~ 57s ago)
   
   === api (Free): npm run server (1)
   api.1: up 2016/09/13 11:52:48 -0700 (~ 35s ago)
   
   ```

5. However, I'm not sure how to actually access the "api" process the way it was supposed to be used, i.e. as a server that handles the `api/food` requests.


## Full repro steps

So how did I even get to this cloned repository in the first place? Here are the full steps:



1. Execute the following commands per "Integrating with a Node Backend" (https://github.com/fullstackreact/food-lookup-demo)
   
   ```
   git clone https://github.com/fullstackreact/food-lookup-demo.git
   cd food-lookup-demo
   npm i
   
   cd client
   npm i
   
   cd ..
   npm start
   ```
2. Navigate to http://localhost:3000/ and type in something like "bread"
3. Go back to the terminal and ctrl-C to kill the server
4. Execute the following commands from "Heroku Buildpack for Create React App" (https://github.com/mars/create-react-app-buildpack)
   ```
   heroku create food-lookup-demo-135 --buildpack https://github.com/mars/create-react-app-buildpack.git
   
   git push heroku master
   ```
   This results in the following error:
   ```
   "remote: npm ERR! Linux 3.13.0-93-generic
   remote: npm ERR! argv "/tmp/build_79e8066fd9274c044b2c2d5355344d08/.heroku/node/bin/node" "/tmp/build_79e8066fd9274c044b2c2d5355344d08/.heroku/node/bin/npm" "run" "build"
   remote: npm ERR! node v5.11.1
   remote: npm ERR! npm  v3.8.6
   remote:
   remote: npm ERR! missing script: build
   remote: npm ERR!"
   ```
5. I don't really understand what's going wrong with the buildpack so I'm going to delete the buildpack:
   ```
   heroku buildpacks:clear
   ```
6. At this point if you run `git push heroku master,` you'll get an error for the web client (defined in the Procfile as "web") saying `"sh: 1: react-scripts: not found"` and `"Local package.json exists, but node_modules missing, did you mean to install?"`

   So I had to add `"install": "cd client && npm i && cd .."` to the scripts section of the server's `/package.json`, like
   
   ```
      "scripts": {
        "install": "cd client && npm i && cd ..",
        "start": "nf start -p 3000",
        "server": "API_PORT=3001 ./node_modules/.bin/babel-node server.js"
      }
   ```
   
   And I also had to make react-scripts a real dependency in the client, i.e. not just a dev dependency:
   
   ```
   client/package.json:
       "dependencies": {
         "react": "^15.2.1",
         "react-dom": "^15.2.1",
         "react-scripts": "0.2.3"
      },
   ```
   
7. Redeploy with the modifications made in step 6:
   ```
   git commit -a -m "."
   git push heroku master
   ```
8. Now it looks like I have to turn on the dyno for the second "api" defined in the Procfile, so I go to the dashboard and do that:
   a. Navigate to https://dashboard.heroku.com/apps/food-lookup-demo-135

   b. See how "api" is turned off -- click "Configure dynos"
   
   c. Click the pencil icon and turn on the "api" dyno (I also had to edit credit card info, though nothing charged)
   
9. OK, now I go back to the terminal and I run:
   ```
   heroku ps
   ```
   
   I see:
   
   ```
   === web (Free): cd client && npm start (1)
   web.1: up 2016/09/13 11:14:03 -0700 (~ 3m ago)
   
   === api (Free): npm run server (1)
   api.1: up 2016/09/13 11:16:01 -0700 (~ 1m ago)
   ```
   

So it looks like I have two processes running, one for the web client and one for the server.

But do I access the server? This is where I'm stuck.
