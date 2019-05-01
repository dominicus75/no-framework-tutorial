[<< előző fejezet](07-inversion-of-control.md) | [következő fejezet >>](09-templating.md)

### Függőség befecskendezés

Egy Dependency Injection Container érzékeli a hívó (kliens) osztály függőségeit és gondoskodik róla, hogy a megfelelő, felparaméterezett szolgáltató objektumot fecskendezze be a kliens példányosításakor.

Alkalmazásunkban az [Auryn](https://github.com/rdlowrey/Auryn) rekurzív függőség-injektort fogjuk alkalmazni, mert ennek fejlesztői mind a dokumentációban, mind a példaprogramokban tudatosan kerülik a sokak szerint ellenjavallt [Service Locator](https://hu.wikipedia.org/wiki/Szolg%C3%A1ltat%C3%A1slok%C3%A1tor) tervezési minta használatát. A Service Locatort azért tartják antipattern-nek mert elrejti a függőségeket és nem tudható, hogy adott kulcs alatt az a típusú szolgáltatás van-e tárolva, amire számítunk.

Telepítsük tehát az Auryn csomagot a megszokott módon, majd hozzunk létre egy új állományt az `src/` könyvtárban, `Dependencies.php` néven. Ebben helyezzük el a következő kódot:

```php
<?php declare(strict_types = 1);

$injector = new \Auryn\Injector;

$injector->alias('Http\Request', 'Http\HttpRequest');
$injector->share('Http\HttpRequest');
$injector->define('Http\HttpRequest', [
    ':get' => $_GET,
    ':post' => $_POST,
    ':cookies' => $_COOKIE,
    ':files' => $_FILES,
    ':server' => $_SERVER,
]);

$injector->alias('Http\Response', 'Http\HttpResponse');
$injector->share('Http\HttpResponse');

return $injector;
```

Mielőtt folytatnánk, vizsgáljuk meg, mit is csinál az Auryn `alias`, `share` és `define` metódusa (*magyarul nem találtam róla leírást, az angol nyelvű dokumentáció [itt](https://github.com/rdlowrey/auryn/blob/master/README.md) található, ebből szemezgettem Patrick Louys leírását kiegészítendő. – a fordító*).

A `share` metódus akkor hasznos, ha ugyan azt az objektumpéldányt szeretnénk az alkalmazás több rétegébe is befecskendezni, ezért nem szeretnénk minden egyes hívásnál másikat létrehozni, hanem a már meglévő megosztására utasítjuk az injektort. Tehát a megosztással végig ugyanaz a példány áll rendelkezésre.

Az `alias` lehetővé teszi, hogy a type hintelésnél osztálynév helyett az interfész nevét használhassuk. Első paramétere ezért az interfész neve, a második pedig az ezt implementáló osztály neve. A kettő egymáshoz rendelésével könnyebbé válik az ugyanazon interfészt megvalósító osztályok esetleges cseréje anélkül, hogy kódunkat más helyen módosítani kellene.

A `define` metódussal tudjuk felparaméterezni a kért szolgáltatást. Első paraméterként a szolgáltatás nevét (amit az `alias` segítségével állítottunk be) várja szövegként (string), második paraméterként pedig egy tömböt, amely a szolgáltató objektum konstruktora által elvárt sorrendben tartalmazza annak paramétereit. A tömb kulcsainak neve elé tett kettőspont (pl. `:get`) azért kell, mert ennek elhagyásával az Auryn alapértelmezésben osztálynévként értelmezi őket.

Ezek után természetesen a `Bootstrap.php` állományunkat is módosítanunk kell. Az eddig közvetlenül példányosított `$request` és `$response` objektumokat  most már az injektorunk fogja létrehozni. Ne feledjük, hogy mindkét objektum példányosítását úgy állítottuk be a `Dependencies.php` idevágó részében, hogy csak egy készüljön belőlük és így ugyanazt az objektumpéldányt használhassuk mindenhol.

Keressük meg az alábbi kódrészt:

```php
$request = new \Http\HttpRequest($_GET, $_POST, $_COOKIE, $_FILES, $_SERVER);
$response = new \Http\HttpResponse;
```
majd cseréljük ki erre:

```php
$injector = include('Dependencies.php');

$request = $injector->make('Http\HttpRequest');
$response = $injector->make('Http\HttpResponse');
```

Az injektor beüzemelése után az útvonal szerinti átirányítást is módosítanunk kell, ezért először keressük meg az alábbi kódrészt:

```php
$class = new $className($response);
$class->$method($vars);
```

majd cseréljük ki erre, hogy immár a vezérlőink példányosítását is az injektorra bízhassuk:

```php
$class = $injector->make($className);
$class->$method($vars);
```

A `Homepage` vezérlőnket pedig az alábbi kóddal módosítsuk:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Request;
use Http\Response;

class Homepage
{
    private $request;
    private $response;

    public function __construct(Request $request, Response $response)
    {
        $this->request = $request;
        $this->response = $response;
    }

    public function show()
    {
        $content = '<h1>Hello World</h1>';
        $content .= 'Hello ' . $this->request->getParameter('name', 'stranger');
        $this->response->setContent($content);
    }
}
```

A `Homepage` vezérlőnknek most kettő függősége van, amelyek feloldásáért az Auryn lett a felelős. Próbáljuk meghívni a nyitólapot a következő GET paraméterekkel: `http://localhost:8000/?name=Arthur%20Dent`. Ha a `Hello World` és a `Hello Arthur Dent` kimenetet kapjuk, akkor minden a legnagyobb rendben, sikeresen lefektettük alkalmazásunk alapjait.

[<< előző fejezet](07-inversion-of-control.md) | [következő fejezet >>](09-templating.md)
