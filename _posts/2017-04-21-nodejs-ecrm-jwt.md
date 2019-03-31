---
layout: post
title: Building a CRM micro-service with JWT authentication in NodeJS
date: 2017-04-21T00:00:00.000Z
categories: blog
---

Customer Relationship Management (CRM) is a central module in every e-commerce application since it manages users (buyers / sellers), organizations, sessions, as well as authentication.
<br>
In this post we will be building a CRM micro-service in NodeJS from scratch and explaining every technical choice on the libraries used.

## Project init
Let's start by creating the basic project structure. Before doing so, you should have NodeJS (version > 6.0.0) as well as NPM (version 3.8.6). If you still do not have these installed check [node's website][nodedownload].
<br>
Start by initializing the node project:

{% highlight sh %}
mkdir skeleton-ecrm-node
cd skeleton-ecrm-node/
npm init
{% endhighlight %} 


NPM will now guide you through the project init, thus setting the project name, author, version, etc.
Here you have an example of how to fill every field:

{% highlight sh %}
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
name: (skeleton-ecrm-node) 
version: (1.0.0) 0.1.
Invalid version: "0.1."
version: (1.0.0) 0.1.0
description: Customer Relationship Management module skeleton in NodeJS
entry point: (index.js) app.js
test command: 
git repository: 
keywords: 
author: 
license: (ISC) 
About to write to /home/nvinuesa/workspace/perso/skeleton-ecrm-node/package.json:

{
  "name": "skeleton-ecrm-node",
  "version": "0.1.0",
  "description": "Customer Relationship Management module skeleton in NodeJS",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this ok? (yes) 

{% endhighlight %} 

[nodedownload]: https://nodejs.org/en/download/

At the end, it will generate a file named ```package.json``` on the project folder with the previously shown contents.

## Http server with express.js

Now that we have or project ready, lets create the main ```app.js``` with the following contents:

{% highlight javascript %}
const express = require('express');

const env = module.exports.env = process.env.NODE_ENV || 'dev';
const conf = require('./config/' + env + '.js');

const logger = require('morgan');
const bodyParser = require('body-parser');
const expressValidator = require('express-validator');
const expressJWT = require('express-jwt');

// MongoDB datasource
const mongo = require('./config/' + env + '.js').mongo;
const mongoose = require('mongoose');
mongoose.connect(mongo.url);
const db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));

// Main Express app
const app = express();
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}));
app.use(expressValidator());
app.use(expressJWT({
	secret: conf.jwt.secret,
	isRevoked: require('./session/session-controller').isRevokedCallback
}).unless({path: '/login'}));
// Routes
const routes = [
	'./profile/profile',
	'./session/session'
].forEach((module) => app.use(require(module + '-routes')));

app.use(function (req, res, next) {
	const err = new Error('Not Found');
	err.status = 404;
	next(err);
});

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
	app.use((err, req, res, next) => {
		console.log(err);
		res.status(err.status || 500);
		res.json({
			code: err.message,
			error: err
		});
	});
}

// production error handler
// no stacktraces leaked to user
app.use((err, req, res, next) => {
	res.status(err.status || 500);
	res.json({
		code: err.message,
		error: {}
	});
});

module.exports = app;

{% endhighlight %} 

Boilerplate code aside, this simple script just declares an express application along with its routing (profile and session routes).
For a more detailed introduction to express.js the reader should follow [this tutorial][expresrouting].
[expresrouting]: https://expressjs.com/en/starter/basic-routing.html

## Profiles CRUD
Now that we have a working http server, let's write our basic profile controller, service and model and routes.
<br>
Our project folder should have the following contents after this section:

{% highlight sh %}
skeleton-ecrm-node
                ├── app.js
                ├── package.json
                └── profile
                         ├── profile-controller.js
                         ├── profile-model.js
                         ├── profile-routes.js
                         └── profile-service.js

{% endhighlight %} 

The profile controller will contain the functions invoked by each route (in profile-routes.js) and that process the http requests.
<br> 
Every business logic should be implemented at the services, which are imported by the controller for processing the results from the database and adding them to the http response object.
<br>
Here is the content of these files:

```profile-controller.js```
{% highlight javascript %}
const Profile = require('./profile-model');
const ProfileService = require('./profile-service');

function requestValidation(req) {

	// Validate the profile received in the body of the request
	req.checkBody('name', 'Profile\'s name can not be empty').notEmpty();
	req.checkBody('email', 'Profile\'s email can not be empty').notEmpty();
	req.sanitize('name').escape();
	req.sanitize('email').escape();
	req.sanitize('name').trim();
	req.sanitize('email').trim();
}

exports.createProfile = function (req, res, next) {

	requestValidation(req);
	const err = req.validationErrors();
	// Check if there are any validation errors in the received profile, else save it
	if (err) {
		return next(err);
	} else {
		const profile = new Profile({
			name: req.body.name,
			email: req.body.email,
			password: req.body.password
		});
		ProfileService.create(profile, (err, profile) => {
			if (err) {
				return next(err);
			}
			res.json(profile);
		});
	}
};

exports.getProfile = function (req, res, next) {

	const id = req.params.id;
	// Validate the request's profile id (must be a valid MongoDB's ObjectID)
	ProfileService.get(id, (err, profile) => {
		if (err) {
			return next(err);
		}
		res.json(profile);
	})
};

exports.getAllProfiles = function (req, res, next) {

	ProfileService.getAll((err, profiles) => {
		if (err) {
			return next(err);
		}
		res.json(profiles);
	})
};

exports.updateProfile = function (req, res, next) {

	requestValidation(req);
	const err = req.validationErrors();
	// Check if there are any validation errors in the received profile, else save it
	if (err) {
		return next(err);
	} else {
		const id = req.params.id;
		const profile = new Profile({
			name: req.body.name,
			email: req.body.email
		});
		ProfileService.update(id, profile, (err) => {
			if (err) {
				return next(err);
			}
			res.json(profile);
		});
	}
};

exports.deleteProfile = function (req, res, next) {

	const id = req.params.id;
	ProfileService.delete(id, (err) => {
		if (err) {
			return next(err);
		}
		res.status(200).end();
	});
};
{% endhighlight %} 

```profile-model.js```
{% highlight javascript %}

const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const Schema = mongoose.Schema;

// Salt constant for the encryption algorithm
const SALT_WORK_FACTOR = 10;

const ProfileSchema = Schema(
	{
		name: {type: String, required: true},
		email: {type: String, required: true},
		password: {type: String, select: false}
	}
);

// Virtual for profile's URL
ProfileSchema
	.virtual('url')
	.get(function () {
		return '/profiles/' + this._id;
	});

// Pre save for the profile schema (encrypt user password)
ProfileSchema
	.pre('save', function (next) {
		const user = this;
		if (!user.password) return next();

		bcrypt.genSalt(SALT_WORK_FACTOR, (err, salt) => {
			if (err) {
				return next(err);
			}
			bcrypt.hash(user.password, salt, (err, hash) => {
				if (err) {
					return next(err);
				}
				user.password = hash;
				next();
			});
		});
	});


//Export model
module.exports = mongoose.model('Profile', ProfileSchema);

{% endhighlight %} 



```profile-routes.js```
{% highlight javascript %}


const express = require('express');
const router = express.Router();

const ProfileController = require('./profile-controller');

// Create profile
router.post('/profiles', ProfileController.createProfile);
// Get profile
router.get('/profiles/:id', ProfileController.getProfile);
// Get all profiles
router.get('/profiles', ProfileController.getAllProfiles);
// Update profile
router.put('/profiles/:id', ProfileController.updateProfile);
// Delete profile
router.delete('/profiles/:id', ProfileController.deleteProfile);

module.exports = router;
{% endhighlight %} 


```profile-service.js```
{% highlight javascript %}

const Profile = require('./profile-model');
const validator = require('validator');
const async = require('async');

function validateProfile(profile) {
	const name = profile.name;
	const email = profile.email;
	// Name must not be empty and must have minimum two words, email must not be empty and be a valid mail address
	return !validator.isEmpty(name)
		&& name.split(' ').length >= 2
		&& !validator.isEmpty(email)
		&& validator.isEmail(email);
}

function invalidId(id) {
	const err = new Error('Profile id ' + id + ' is not valid!');
	err.status = 404;
	return err;
}

function invalidEmail(email) {
	const err = new Error('Profile email ' + email + ' is not valid!');
	err.status = 409;
	return err;
}

function profileNotFound(id) {
	const err = new Error('Profile ' + id + ' not found!');
	err.status = 404;
	return err;
}

exports.create = function (profile, next) {

	if (!validateProfile(profile)) {
		const err = new Error('Invalid profile format.');
		err.status = 409;
		return next(err)
	}
	async.waterfall([
			function (callback) {
				Profile.count({email: profile.email}, callback);
			},
			function (count, callback) {
				if (count !== 0) {
					const err = new Error('Profile with email ' + profile.email + ' already exists!');
					err.status = 409;
					return callback(err);
				}
				callback();
			},
			function (callback) {
				profile.save((err, profile) => {
					callback(err, profile)
				})
			}
		],
		function (err, profile) {
			return next(err, profile);
		}
	);
};

exports.get = function (id, next) {

	// Validate the request's profile id (must be a valid MongoDB's ObjectID)
	if (id.match(/^[0-9a-fA-F]{24}$/)) {
		Profile.findById(id)
			.exec((err, profile) => {
				if (err) {
					return next(err);
				}
				if (!profile) {
					return next(profileNotFound(id));
				}
				// Successful, retrieve profile (with null error)
				return next(null, profile);
			});
	} else {
		return next(invalidId(id));
	}
};

exports.getByEmail = function (email, next) {

	// Validate the request's profile email
	if (validator.isEmail(email)) {
		Profile.findOne({email: email})
			.exec((err, profile) => {
				if (err) {
					return next(err);
				}
				if (!profile) {
					return next(profileNotFound(email));
				}
				// Successful, retrieve profile (with null error)
				return next(null, profile);
			});
	} else {
		return next(invalidEmail(email));
	}
};

exports.getWithPasswordByEmail = function (email, next) {

	// Validate the request's profile email
	if (validator.isEmail(email)) {
		Profile.findOne({email: email})
			.select('+password')
			.exec((err, profile) => {
				if (err) {
					return next(err);
				}
				if (!profile) {
					return next(profileNotFound(email));
				}
				// Successful, retrieve profile (with null error)
				return next(null, profile);
			});
	} else {
		return next(invalidEmail(email));
	}
};

exports.getAll = function (next) {

	Profile.find()
		.exec((err, profiles) => {
			if (err) {
				return next(err);
			}
			// Successful, retrieve all profile (with null error)
			return next(null, profiles);
		});
};

exports.update = function (id, profile, next) {

	// Validate the request's profile id (must be a valid MongoDB's ObjectID)
	if (id.match(/^[0-9a-fA-F]{24}$/)) {
		if (!validateProfile(profile)) {
			const err = new Error('Invalid profile format.');
			err.status = 409;
			return next(err)
		}
		// Update the profile
		Profile.findByIdAndUpdate(id, {name: profile.name, email: profile.email})
			.exec((err, profile) => {
				if (err) {
					return next(err);
				}
				if (!profile) {
					return next(profileNotFound(id));
				}
				return next();
			});
	} else {
		return next(invalidId(id));
	}
};

exports.delete = function (id, next) {

	// Validate the request's profile id (must be a valid MongoDB's ObjectID)
	if (id.match(/^[0-9a-fA-F]{24}$/)) {
		// Delete the profile
		Profile.findByIdAndRemove(id)
			.exec((err, profile) => {
				if (err) {
					return next(err);
				}
				if (!profile) {
					return next(profileNotFound(id));
				}
				return next();
			});
	} else {
		return next(invalidId(id));
	}
};

{% endhighlight %} 

