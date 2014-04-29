## Node Rollout
Feature rollout management for Node.js built on Redis

``` sh
npm install node-rollout --save
```

``` js
// configuration.js
var rollout = require('node-rollout')
rollout.handler('new_homepage', {
  // 1% of regular users
  id: {
    percentage: 1
  },
  // All users with the company email
  employee: {
    percentage: 100,
    condition: function (val) {
      return val.match(/@company-email\.com$/)
    }
  },
  // 50% of users in San Francisco
  geo_sf: {
    percentage: 50,
    condition: function (val) {
      return geolib.getDistance([val.lat, val.lon], [37.768, -122.426], 'miles') < 7
    }
  }
})
```

``` js
// A typical Express app
...
require('./configuration')

app.get('/', new_homepage, old_homepage)

function new_home_page(req, res, next) {
  rollout.get('new_homepage', current_user.id, {
    employee: current_user.email,
    geo: [current_user.lat, current_user.lon]
  })
    .then(function () {
      res.render('home/new-index')
    })
    .otherwise(next)
}

function old_home_page (req, res, next) {
  res.render('home/index')
}

```


### API Options

#### `rollout.get(key, uid, values)`

 - `key`: `String` The rollout feature key. Eg "new_homepage"
 - `uid`: `String` The identifier of which will determine likelyhood of falling in rollout. Typically a user id.
 - `values`: `Object` A lookup object with default percentages and conditions

#### `rollout.handler(key, flags)`
 - `key`: `String` The rollout feature key
 - `flags`: `Object`
  - `flagname`: `String` The name of the flag. Typically `id`, `employee`, `ip`, or any other arbitrary item you would want to modify the rollout
    - `percentage`: `NumberRange` from 0 - 100. Can be set to a third decimal place such as `0.001` or `99.999`. Or simply `0` to turn off a feature, or `100` to give a feature to all users
    - `condition`: `Function` a white-listing method by which you can add users into a group. See examples.

#### `rollout.update(key, flags)`
 - `key`: `String` The rollout feature key
 - `flags`: `Object` mapping of `flagname`:`String` to `percentage`:`Number`

example

``` js
rollout.update('new_homepage', {
  id: 33.333,
  employee: 50,
  geo_sf: 25
})
```