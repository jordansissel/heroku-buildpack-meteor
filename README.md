# Heroku buildpack for meteor

## Usage

```
% heroku create --buildpack https://github.com/jordansissel/heroku-buildpack-meteor.git
```

## Example

Create a sample app with 'meteor'

```
% meteor create --example todos
todos: created.

To run your new app:
   cd todos
   meteor
```

Put it in git.

```
% cd todos
% git init
Initialized empty Git repository in /tmp/a/todos/.git/
% git add .
% git commit -m "Sample todos app!"
```

Create your heroku app

```
% heroku create --buildpack https://github.com/jordansissel/heroku-buildpack-meteor.git
```

Or if your Heroku app already exists

```
% heroku buildpacks:set https://github.com/jordansissel/heroku-buildpack-meteor.git
```

Configure your plugins & settings

```
% heroku addons:create mongolab:sandbox  # optional: use whatever mongo provider you prefer
% heroku config:add MONGO_URL=<insert_value_of_MONGOLAB_URI_here>
% heroku config:add ROOT_URL=<insert_url_created_above_here>
```

Add [session affinity](https://devcenter.heroku.com/articles/session-affinity) so your app will still work with more than one dyno
```
% heroku labs:enable http-session-affinity
```

Optional step, if you are using a ```settings.json``` file to configure your Meteor application

```
% heroku config:add METEOR_SETTINGS="$(cat settings.json)"
```

Deploy it

```
% git push heroku master
```

Enjoy!
