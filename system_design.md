# Django React.js Primer

`First thing first - shout out to @mbrochh for the awesome templates and inspiring me to work on this. `

A primer for building anything with [Django](https://www.djangoproject.com) and
[React](http://facebook.github.io/react/).

---


### The Stack

Our stack usually consists of the following tools:

[nginx](http://nginx.org), [uwsgi](http://uwsgi-docs.readthedocs.org/en/latest/),
[Django](https://www.djangoproject.com), [django-cms](https://www.django-cms.org),
[django-filer](http://django-filer.readthedocs.org/en/latest/),
[django-debug-toolbar](http://django-debug-toolbar.readthedocs.org),
[Bootstrap](http://getbootstrap.com), [less](http://lesscss.org),
[Fabric](http://www.fabfile.org), [Memcached](http://www.memcached.org) and
[solr](http://lucene.apache.org/solr/)

All the above is essentially wrapped up in the following boilerplate
repositories:

[django-project-template](https://github.com/bitmazk/django-project-template),
[django-reusable-app-template](https://github.com/bitmazk/django-reusable-app-template) and
[django-development-fabfile](https://github.com/bitmazk/django-development-fabfile)
[ansible-ubuntu-django](https://github.com/bitmazk/ansible-ubuntu-django)

### Project Structure

We try to stick to the DRY principle as best as possible, even across different
projects. Therefore, we release almost everything as reusable Django apps on
Github and re-use apps in all our projects whenever possible.

A project usually only consists of the following major parts:

1. A settings module which has a lot of reusable apps in the `INSTALLED_APPS`
   setting.
2. A `urls.py` which is hooks up all those reusable apps.
3. A `requirements.txt` file which allows new developers to setup a local
   development environment quickly.
4. A fabfile which allows developers to setup a local database, run the
   Python unit-tests, measure code coverage, lint the code and run
   automated deployments.
5. A templates folder which contains a `base.html` and lots of overrides for
   all the templates of all reusable apps.
6. A static folder which contains the bootstrap styles, our own style
   overrides, images, third party .css and .js files, usually for jQuery
   plugins.

### Shortcomings

The above stack and project structure works extremely well for our small team.
Everyone knows about this stack and structure very well and therefore everyone
is able to quickly setup any project and commit fixes.

We consider our products to be extremely well crafted and highly maintainable
but as with most projects, there are some shortcomings that we have realized
over time but never really cared to solve (because things work good enough).



## Possible Solution

I will try to re-work this chapter soon. I have recently created this
[Django ReactJS Boilerplate](https://github.com/mbrochh/django-reactjs-boilerplate)
which finally emerged as a way forward for me to use ReactJS in all my future
Django projects. I still need to update that boilerplate with
django-rest-framework and the usual common tasks like authentication, making
GET, POST, PUT requests and hopefully server side rendering at the very end.

Once that boilerplate is in place and I have made up my mind about my best
practices that are worth sharing, I will put them into this chapter.

Everything that follows is pretty old and outdated and was based on my past self
that had no idea what was going on. Handle with care :)

### New Stack

#### For New Projects

Django will no longer serve any templates. It will only provide a JSON API via
[django-rest-framework](http://www.django-rest-framework.org). Spike Brehm
of airbnb has shared some nice ideas about
[isomorphic javascript](http://nerds.airbnb.com/isomorphic-javascript-future-web-apps/).
If you scroll to the second image of that post, you will get a good idea on
how the new stack should work. Admins will still be able to use the awesome
Django admin in order to maintain the database and we will still be able to
use the awesome ORM (and it's caching solutions) to make queries that any
developer can easily understand, but we will no longer use Django's views,
forms and templating language. To be fair, the APIViews of
django-rest-framework are similar to Django's views and forms, so we are really
just giving up the templating engine.

[Node.js](https://nodejs.org) will serve the whole bundled app to the client.
This is the isomorphic part. We will have a `server.js` file which consists
of an [express](http://expressjs.com) application that listens to the requests
and responds with the matching rendered React component. This way, we can
pre-render the requests on the server, so that search engines won't get an
empty page. Once the client recieves the response, everything else happens on
the client side (just calling API endpoints, re-rendering parts of the page).

Nginx will serve the Django API at `example.com/api/v1/` (via uwsgi) and the
Node.js server at `example.com`. Nginx will also serve static files (as usual).

[react-router](https://github.com/rackt/react-router) will be used as a
replacement of our beloved Django `urls.py` files. It defines which URL should
map to which React component and has some cool ways to transition between
"views", handle redirects and even pickup interrupted intents (i.e. when you
try to access a restricted area and need to login first).

[react-bootstrap](http://react-bootstrap.github.io/index.html) shall be used
to replace vanilla bootstrap.

[newforms-bootstrap](https://github.com/insin/newforms-bootstrap) could be used
to render forms. It has an API that was inspired by Django, so the code for
this will look very familiar to us. It will handle frontend-validation nicely.
Unfortunately, we will still have to define the same validation in
django-rest-framework (always validate on the server), so there is a possible
duplication of code. It might be a better idea to always let
django-rest-framework do the validation and then just display the returned
error messages (if any).

[Radium](https://github.com/FormidableLabs/radium) will be used to create
styles right there in the files where the components are implemented. By also
passing in styles via props, this would allow us to override the implemented
default styles by passing in a big StyleSheet into the ContainerComponent.
ReactNative uses something with the same syntax. I'm not sure if this would be
compatible with newforms-bootstrap, though and I'm also not sure if this is
a smart idea at all. It would probably be better to just override the
bootstrap styles in the same old-fashioned way we always did but make sure that
*every* style refers to a React component name.

[Webpack](http://webpack.github.io) will be used to bundle the app. It should
result in one or more vendors files (i.e. for react.js itself) and one file
that contains our own whole app. If the total size of the bundle would be
somewhere at 200kb, I guess everything could just be included into one big bundle.

#### For Existing Projects

React.js could be added as a dependency like jQuery and
[django-react](https://github.com/markfinger/django-react) could be used to
render components on the server and passing them into the template context.

If the components live behind a login-wall and cannot reached by crawlers,
django-react would even not be necessary at all, the user would simply see
a short loading spinner before the component comes to life.

Not sure, yet, if and how Webpack could be used to bundle all components that
might be spread throughout different corners of the project. Ideally we would
include React.js, our components bundle and then initialize whatever component
we want to use in `<script>` tags in the templates where they are used (just
like we do it today with jQuery plugins).

### New Project Structure

Repositories for new projects should have two main folders: One for the Django
API and one for the React frontend. I'm not sure if fabfiles, gulp configs and
Webpack configs should also live in the project root or within the two main
folders. Anyone working on the project would always have to start all servers
and file system watchers, so having all the glue-scripts in the root folder
is probably the way to go.

## The Toolchain

## License

<img align="right" src="http://opensource.org/trademarks/opensource/OSI-Approved-License-100x137.png">

The class is licensed under the [MIT License](http://opensource.org/licenses/MIT):

Copyright &copy; 2018 [Chris Ohk](http://www.github.com/utilForever).

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

### Isomorphic Setup

https://github.com/kriasoft/react-starter-kit
https://github.com/jesstelford/react-isomorphic-boilerplate
https://github.com/gpbl/isomorphic-react-template?files=1
https://gist.github.com/mitchelkuijpers/11281981
http://fluxible.io/api/bringing-flux-to-the-server.html
https://github.com/yahoo/flux-examples/tree/master/react-router
https://github.com/koba04/react-boilerplate/blob/master/app/routes.js
https://github.com/sogko/gulp-recipes/blob/master/browser-sync-nodemon-expressjs/gulpfile.js
https://gist.github.com/dstroot/22525ae6e26109d3fc9d


