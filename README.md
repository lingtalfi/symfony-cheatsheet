Symfony Cheatsheet
=====================
2016-07-24



Run the server
------------------
```bash
php bin/console server:run
```


Important paths
------------------

```
- templates: app/Resources/views
- controllers: src/AppBundle/Controller
----- assets: Resources/public
----- templates: Resources/views
```


Dump the config of a bundle
---------------------
```bash
 php bin/console config:dump-reference framework
```



Routes
--------------------

### Visualizing routes/debugging

```bash
php bin/console debug:router
php bin/console debug:router article_show
php bin/console router:match /blog/my-latest-post
```

### Annotations

```php
/**
 * @Route("/hello/{name}", name="hello")
*/
/**
 * @Route("/blog/{page}", defaults={"page" = 1})
 */
 /**
 * @Route("/blog/{page}", defaults={"page": 1}, requirements={
 *     "page": "\d+"
 * })
 */
 /**
  * @Route("/{_locale}", defaults={"_locale": "en"}, requirements={
  *     "_locale": "en|fr"
  * })
  */

	/**
	 * @Route("/api/posts/{id}")
	 * @Method({"GET","HEAD"})
	 */  
```

### using yaml
```yaml
blog:
    path:      /blog/{page}
    defaults:
        _controller: AppBundle:Lucky:index
        page:        1
        title:       "Hello world!"

contact:
    path:     /contact
    defaults: { _controller: AcmeDemoBundle:Main:contact }
    condition: "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'"


# pointing to a controller with annotations
app:
    resource: '@AppBundle/Controller/'
    type:     annotation 


# include other routing config files
app2:
    resource: '@AcmeOtherBundle/Resources/config/routing.yml'

# use a prefix
app3:
    resource: '@AppBundle/Controller/'
    type:     annotation
    prefix:   /site

```


### Generating URLs

```php
$params = $this->get('router')->match('/blog/my-blog-post');
// array(
//     'slug'        => 'my-blog-post',
//     '_controller' => 'AppBundle:Blog:show',
// )

$uri = $this->get('router')->generate('blog_show', array(
    'slug' => 'my-blog-post'
));
// /blog/my-blog-post

$url = $this->generateUrl(
    'blog_show',
    array('slug' => 'my-blog-post')
);


// absolute
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

$this->generateUrl('blog_show', array('slug' => 'my-blog-post'), UrlGeneratorInterface::ABSOLUTE_URL);
// http://www.example.com/blog/my-blog-post
```

For ajax requests, we can use the [FOSJsRoutingBundle](https://github.com/FriendsOfSymfony/FOSJsRoutingBundle).

```js
var url = Routing.generate(
    'blog_show',
    {'slug': 'my-blog-post'}
);
```

From a twig template

```twig
<a href="{{ path('blog_show', {'slug': 'my-blog-post'}) }}">
  Read this blog post.
</a>

<!-- absolute -->
<a href="{{ url('blog_show', {'slug': 'my-blog-post'}) }}">
  Read this blog post.
</a>
```








Special routing params/vars
-----------------

- _controller
- _format
- _locale
- _route




```php
// ...
class LuckyController extends Controller
{
    
    public function indexAction($title)
    {
        return new Response($title);
    }
}    
```


Controller naming patterns
-----------------------------
http://symfony.com/doc/current/book/routing.html#controller-naming-pattern

- _controller: bundle:controller:action
- _controller: AppBundle:Blog:show
- _controller: AppBundle\Controller\BlogController::showAction
- _controller: service_name:indexAction




Templates
---------------

### how to reference templates
```php
<?php

namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class LuckyController extends Controller
{
    /**
     * @Route("/doo")
     */
    public function indexAction()
    {


        /**
         * Using the filepath notation
         */
        // both look in
        // app/Resources/views/test.html.twig
        return $this->render('::test.html.twig');
        return $this->render('test.html.twig');

        /**
         * Using the @ prefix
         */
        // look in
        // src/AppBundle/Resources/views/test.html.twig
        return $this->render('@App/test.html.twig');


        /**
         * Using the bundle:controller:filepath string syntax
         */
        // look in
        // - app/Resources/AcmeBlogBundle/views/Blog/index.html.twig
        // - src/Acme/BlogBundle/Resources/views/Blog/index.html.twig
        return $this->render('AcmeBlogBundle:Blog:index.html.twig', array('blogs' => $blogs));


        // look in
        // - app/Resources/AppBundle/views/Lucky/test.html.twig
        // - src/AppBundle/Resources/views/Lucky/test.html.twig
        return $this->render('AppBundle:Lucky:test.html.twig');

        // look in
        // - app/Resources/AppBundle/views/test.html.twig
        // - src/AppBundle/Resources/views/test.html.twig
        return $this->render('AppBundle::test.html.twig');


    }
}

```




Copy all your bundles assets to the web directory
-------------------------------------
(actually create symlinks...)

```bash
php bin/console assets:install web --symlink
```




Twig mini cheatsheet
-------------------------

- var/cache/{environment}/twig
- provides app variable (contains user, request, session, environment, debug)

```twig
{% for i in 0..10 %}
    <div class="{{ cycle(['odd', 'even'], i) }}">
      <!-- some HTML here -->
    </div>
{% endfor %}


<ul>
    {% for user in users if user.active %}
        <li>{{ user.username }}</li>
    {% else %}
        <li>No users found</li>
    {% endfor %}
</ul>



{% block sidebar %}
    <h3>Table of Contents</h3>

    {# ... #}

    {{ parent() }}
{% endblock %}
```







Services
-------------
```php
$templating = $this->get('templating');
$router = $this->get('router');
$mailer = $this->get('mailer');
```

```bash
php bin/console debug:container
```

Pages not found
------------------

```php
if (!$product) {
	// extends NotFoundHttpException
	throw $this->createNotFoundException('The product does not exist');
}
```





Flash messages
--------------------
```php
class LuckyController extends Controller
{

    public function indexAction()
    {
        $this->addFlash(
            'notice',
            'Your changes were saved!'
        );
        return new Response("blabla1");
    }

    public function index2Action()
    {
        return $this->render('lucky/flash.html.twig');
    }    
}
```

```twig
<!-- lucky/flash.html.twig -->
{% for flash_message in app.session.flashBag.get('notice') %}
    <div class="flash-notice">
        {{ flash_message }}
    </div>
{% endfor %}
```

Controller Response
----------------------
### json
```php
// ...
public function indexAction()
{
    // returns '{"username":"jane.doe"}' and sets the proper Content-Type header
    return $this->json(array('username' => 'jane.doe'));

    // the shortcut defines three optional arguments
    // return $this->json($data, $status = 200, $headers = array(), $context = array());
}
```


Other interesting subjects
-------------------------

### controllers

http://symfony.com/doc/current/book/controller.html

- Creating static pages without even creating a controller (only route and template needed)





