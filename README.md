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
...
