[<< előző fejezet](09-templating.md) | [következő fejezet >>](11-page-menu.md)

### Dinamikus oldalak

Eddig csupán egy minimális funkcionalitású statikus oldallal lettünk gazdagabbak. Lépjünk is túl a nem kimondottan fantáziadús hello world példán, és adjunk némi valódi funkcionalitást az alkalmazásunknak.

Első funkciónk [markdown](https://szit.hu/doku.php?id=oktatas:web:markdown) állományokból történő dinamikus oldalak generálása lesz (*Fordítói megjegyzés: a szerző az **oldal** kifejezést a Drupalból vagy a WordPress-ből ismert [jelentéstartalommal](http://wphu.org/konyv/2-2-oldalak/) használja, tehát egy tartalomtípust ért alatta, nem úgy általában egy weboldalt*).

Ehhez hozzuk létre a `Page` vezérlőt az alábbi tartalommal:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

class Page
{
    public function show($params)
    {
        var_dump($params);
    }
}
```
Ha ezzel megvagyunk, adjunk hozzá egy új route-ot is az `src/Routes.php` állományban:

```php
['GET', '/{slug}', ['Example\Controllers\Page', 'show']],
```

Most próbáljuk ki a művünket és hívjuk meg néhány tetszőlegesen kiválasztott url-el az alkalmazásunkat, például a `http://localhost:8000/test` vagy a `http://localhost:8000/hello` éppen megfelel. Amint láthatjuk, minden alkalommal a `Page` vezérlőt hívtuk meg, a `$params` tömbben átadva neki a az oldal "[szép url](http://webmestertanfolyam.hu/webmester-blog/szep-url-generalas)"-jét (*A fordító megjegyzése: az eredeti szövegben itt a **slug** kifejezés szerepel, aminek a jelentését – elterjedt fordítás hiányában – szép vagy keresőbarát url-ként lehet legjobban visszaadni, ha esetleg a Gugli Fordító által felajánlott **meztelen csiga** nem nyeri el a tetszésünket...*).

Kezdetnek hozzunk létre néhány oldalt. Még nem fogunk adatbázist használni, ezért most létrehozunk egy új mappát a projektünk gyökerében, amit `pages`-nek fogunk hívni. Ebben a könyvtárban fogjuk elhelyezni az `.md` kiterjesztésű szöveges állományainkat, amelyek oldalaink tartalmát lesznek hivatva tárolni. Először hozzunk létre egy `page-one.md` elnevezésű fájlt, az alábbi kifejező tartalommal:

```
Ez itt egy oldal.
```

Now we will have to write some code to read the proper file and display the content. It might seem tempting to just put all that code into the `Page` controller. But remember [Separation of Concerns](http://en.wikipedia.org/wiki/Separation_of_concerns). There is a good chance that we will need to read the pages in other places in the application as well (for example in an admin area).

So let's put that functionality into a separate class. There is a good chance that we might switch from files to a database later, so let's use an interface again to make our page reader decoupled from the actual implementation.

In your 'src' folder, create a new folder `Page`. In there we will put all the page related classes. Add a new file in there called `PageReader.php` with this content:

```php
<?php declare(strict_types = 1);

namespace Example\Page;

interface PageReader
{
    public function readBySlug(string $slug) : string;
}
```

For the implementation, create a `FilePageReader.php` file. The file will looks like this:

```php
<?php declare(strict_types = 1);

namespace Example\Page;

use InvalidArgumentException;

class FilePageReader implements PageReader
{
    private $pageFolder;

    public function __construct(string $pageFolder)
    {
        $this->pageFolder = $pageFolder;
    }

    public function readBySlug(string $slug) : string
    {
        return 'I am a placeholder';
    }
}
```

As you can see we are requiring the page folder path as a constructor argument. This makes the class flexible and if we decide to move files or write unit tests for the class, we can easily change the location with the constructor argument.

You could also put the page related things into it's own package and reuse it in different applications. Because we are not tightly coupling things, things are very flexible.

This will do for now. Let's create a template file for our pages with the name `Page.html` in the `templates` folder. For now just add `{{ content }}` in there.

Add the following to your `Dependencies.php` file so that the application know which implementation to inject for our new interface. We also define the the `pageFolder` there.

```php
$injector->define('Example\Page\FilePageReader', [
    ':pageFolder' => __DIR__ . '/../pages',
]);

$injector->alias('Example\Page\PageReader', 'Example\Page\FilePageReader');
$injector->share('Example\Page\FilePageReader');
```


Now go back to the `Page` controller and change the `show` method to the following:

```php
public function show($params)
{
    $slug = $params['slug'];
    $data['content'] = $this->pageReader->readBySlug($slug);
    $html = $this->renderer->render('Page', $data);
    $this->response->setContent($html);
}
```

To make this work, we will need to inject a `Response`, `Renderer` and a `PageReader`. I will leave this to you as an exercise. Remember to `use` all the proper namespaces. Use the `Homepage` controller as a reference.

Did you get everything to work?

If not, this is how the beginning of your controller should look now:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Response;
use Example\Template\Renderer;
use Example\Page\PageReader;

class Page
{
    private $response;
    private $renderer;
    private $pageReader;

    public function __construct(
        Response $response,
        Renderer $renderer,
        PageReader $pageReader
    ) {
        $this->response = $response;
        $this->renderer = $renderer;
        $this->pageReader = $pageReader;
    }
...
```

So far so good, now let's make our `FilePageReader` actually do some work.

We need to be able to communicate that a page was not found. For this we can create a custom exception that we can catch later. In your `src/Page` folder, create a `InvalidPageException.php` file with this content:

```php
<?php declare(strict_types = 1);

namespace Example\Page;

use Exception;

class InvalidPageException extends Exception
{
    public function __construct($slug, $code = 0, Exception $previous = null)
    {
        $message = "No page with the slug `$slug` was found";
        parent::__construct($message, $code, $previous);
    }
}
```

Then in the `FilePageReader` file add this code at the end of your `readBySlug` method:

```php
$path = "$this->pageFolder/$slug.md";

if (!file_exists($path)) {
    throw new InvalidPageException($slug);
}

return file_get_contents($path);
```

Now if you navigate to a page that does not exist, you should see an `InvalidPageException`. If a file exists, you should see the content.

Of course showing the user an exception for an invalid URL does not make sense. So let's catch the exception and show a 404 error instead.

Go to your `Page` controller and refactor the `show` method so that it looks like this:

```php
public function show($params)
{
    $slug = $params['slug'];

    try {
        $data['content'] = $this->pageReader->readBySlug($slug);
    } catch (InvalidPageException $e) {
        $this->response->setStatusCode(404);
        return $this->response->setContent('404 - Page not found');
    }

    $html = $this->renderer->render('Page', $data);
    $this->response->setContent($html);
}
```

Make sure that you use an `use` statement for the `InvalidPageException` at the top of the file.

Try a few different URLs to check that everything is working as it should. If something is wrong, go back and debug it until it works.

Ahogy mindig, most se felejtsünk el commitolni!

[<< előző fejezet](09-templating.md) | [következő fejezet >>](11-page-menu.md)
