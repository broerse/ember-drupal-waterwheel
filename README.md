![Waterwheel Ecosystem](https://raw.githubusercontent.com/acquia/waterwheel-js/assets/waterwheel.png)

# Ember-Drupal Waterwheel

The intent of this addon is to give Ember developers a set of tools to quickly integrate with
a Drupal site for use as Ember Data's backend. It provides blueprints to help you get your Ember application connected
with a Drupal website as the data/content backend. You can start a new Ember app from scratch and add
these blueprints from the start, or add them to an existing Ember app.

**Also check out [ember-waterwheel-app](https://github.com/acquia/ember-waterwheel-app)
for a reference application that illustrates what can be easily built using this addon, as well as demonstrating
how to modify entities on your Drupal backend from an Ember app.**

---

## Requirements

* [Git](https://git-scm.com/)
* [Node.js and npm](https://nodejs.org/)
* [Bower](https://bower.io/)
* [Ember CLI](https://ember-cli.com/)
* [PhantomJS](http://phantomjs.org/) (only required for automated tests)
* A [Drupal 8](https://www.drupal.org) site with your content

## Installation

If you have not yet created an Ember application, generate a new one:

* `ember new your-new-app-name`

and remove the default welcome message by removing these lines from `app/templates/application.hbs`:
```handlebars
{{!-- The following component displays Ember's default welcome message. --}}
{{welcome-page}}
{{!-- Feel free to remove this! --}}
```

Install this addon:

* `ember install ember-drupal-waterwheel`

You'll see a series of prompts asking which built-in Drupal entities, and other features like OAuth 2.0 authentication,
you'd like to install.

The addon will add a few required libraries to your project, as well as Adapter and Serializer classes that customize
JSON API communication to be compatible with the Drupal backend. If you choose to use OAuth 2.0, an Authenticator,
Authorizer, and Session Service are also created to manage OAuth2. Finally, a "user" model is created to store user
entities fetched from the Drupal backend (which is necessary to support authentication, and to represent user
ownership of entities.)

Some configuration settings will be added to your app's `config/environment.js` file. These will need
to be customized to match the base URL of your Drupal backend site and your OAuth settings (see
[Drupal Authentication Using OAuth 2.0](#drupal-authentication-using-oauth-2.0)). You'll also see configuration for
`ENV.drupalEntityModels`, where you specify which Drupal entities you'll be using from the Ember app. Search for
"@todo" to find any items that will need to be customized for your application.

## Drupal Site Configuration

1. Your Drupal backend needs to have the JSON API contrib module enabled. From your Drupal root, run:
    * `composer require "drupal/jsonapi" "drupal/simple_oauth:~2.0" -o`
    * `drush en -y jsonapi`

1. Cross-Origin Resource Sharing (CORS) needs to be enabled on the Drupal site. Modify the related lines near the
bottom of your `sites/default/services.yml` file:

```yaml
cors.config:
  enabled: true
  # Specify allowed headers, like 'x-allowed-header'.
  allowedHeaders: ['Content-Type', 'Access-Control-Allow-Headers', 'Authorization']
  # Specify allowed request methods, specify ['*'] to allow all possible ones.
  allowedMethods: ['POST', 'GET', 'OPTIONS', 'PATCH', 'DELETE']
  # Configure requests allowed from specific origins.
  allowedOrigins: ['*']
  # Sets the Access-Control-Expose-Headers header.
  exposedHeaders: true
  # Sets the Access-Control-Max-Age header.
  maxAge: false
  # Sets the Access-Control-Allow-Credentials header.
  supportsCredentials: true
```

To increase security, the `allowedOrigins` setting should be changed to a list of permitted origins from which
your app will be served, such as:
```yaml
  allowedOrigins: ['localhost:4200', 'localhost:3000', 'yourbackendsite.com']
```

### Drupal Authentication Using OAuth 2.0

1. If you don't need to perform authentication to your Drupal backend, you're done configuring Drupal! Otherwise,
you'll also need to enable and configure the Simple OAuth contrib module:
    * `composer require "drupal/simple_oauth:~2.0" -o`
    * `drush en -y simple_oauth`

1. Generate encryption keys for Simple OAuth:
    * `openssl genrsa -out private.key 2048`
    * `openssl rsa -in private.key -pubout > public.key`

1. On your Drupal site, browse to `/admin/config/people/simple_oauth` and enter the full paths to both encryption keys
in the Simple OAuth Settings.

1. Browse to `/admin/config/people/simple_oauth/oauth2_client/add` to create OAuth client settings for this app. For
"Label", specify something like "Ember Waterwheel app". Leave all other settings at their defaults and click "Save".
Copy the UUID from the list of OAuth clients and paste it into this application's `config/environment.js` at the location
marked with a @todo. Also fill in the Drupal site's URL in that same file.

## Using Custom Drupal Entities

You can use custom Drupal entities in your Ember app by creating the necessary Ember model, template, and route. A
blueprint is provided by this addon to make the process easier.

1. Generate boilerplate model, route, and template files for your custom entity/content type (a one-word, simplified `entity_name` is
highly recommended):
    * `ember generate drupal-entity entity_name`

1. Adjust the custom entity's model as needed to include all fields you're interested in from the Drupal entity

1. Configure mapping of the Drupal entity to the Ember model in `config/environment.js` (see @todo)

1. Fill in the custom entity's template file to describe how the entity should be displayed

See the "article" model, route, and template files from [ember-waterwheel-app](https://github.com/acquia/ember-waterwheel-app)
for a more detailed example of how these files might be implemented.

## Using Drupal Built-in Entities

If you'd like to use Drupal's built-in entities (article, tag, and file), but you didn't choose to install these at the
time you installed this addon, you can manually install the necessary model, template, and route for these entities:
* `ember generate drupal-article`
* `ember generate drupal-tag`
* `ember generate drupal-file`

You can install the files for any, or all, of these entities as needed. From here, you can edit the Ember template
files for these models as you typically would using
the Ember CLI. It's also a good idea to search for "@todo" in the generated files, to see if there
are customizations or coding choices you can make.

## Running / Development

During development, you can build and serve your Ember app to a browser by running:

* `ember serve`
* Visit your app at [http://localhost:4200](http://localhost:4200).

For running in production, you'll want to do some research. Some helpful resources to get started:
[Ember CLI deploy resources](http://ember-cli-deploy.com/resources/)

### Running Using FastBoot (Server-side Rendering) [Experimental]

Ember provides the FastBoot addon for server-side rendering. Server-side redering has the advantages of allowing your
app to work on browsers that don't have JavaScript enabled, and can also help many search engines to index your app's
content. It may also speed up initial page load times. **(Note that FastBoot is considered experimental until it reaches
its 1.0 release, and as such is not guaranteed to work!)**

To use FastBoot in your Ember app, you'll first need to install the addon:

* `ember install ember-cli-fastboot`

FastBoot has some built-in security measures that require a list of the domains you'll be serving your app from the be
whitelisted. This can be done in your app's `config/environment.js` file by adding the following lines inside the
`var ENV = { ... }` definition:

```javascript
fastboot: {
  hostWhitelist: ['yourbackendsite.com', 'yourstagingsite.com', /^localhost:\d+$/]
}
```
This will whitelist serving the app on your production and staging environments, as well as locally at any port.

If you're using this addon's OAuth 2.0 features, you'll need to switch the session store to use cookies instead of
browser local storage. Do this by creating `app/session-stores/application.js` with the following two lines:

```javascript
import CookieStore from 'ember-simple-auth/session-stores/cookie';
export default CookieStore.extend();
```

Finally, whitelist a library required for ember-simple-auth to work with FastBoot by adding these lines at the bottom
of your app's package.json file:

```javascript
  "fastbootDependencies": [
    "node-fetch"
  ]
```

Now, you can start serving your app with FastBoot:

* `ember fastboot --serve-assets`
* Visit your app at [http://localhost:3000](http://localhost:3000).

### Running Tests

* `npm test` (Runs `ember try:each` to test your addon against multiple Ember versions)
* `ember test`
* `ember test --server`

### Building

* `ember build` (development)
* `ember build --environment production` (production)

## Further Reading / Useful Links

* [ember.js](http://emberjs.com/)
* [ember-cli](https://ember-cli.com/)
* [Ember Data](https://github.com/emberjs/data)
* [Drupal JSON API module](https://www.drupal.org/project/jsonapi/)
* [ember-waterwheel-app](https://github.com/acquia/ember-waterwheel-app)
* [JSON API specification](http://jsonapi.org/format/)
* Ember development browser extensions:
  * [Ember Inspector for Chrome](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi)
  * [Ember Inspector for Firefox](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/)
