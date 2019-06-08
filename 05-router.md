[<< előző fejezet](04-http.md) | [következő fejezet >>](06-dispatching-to-a-class.md)

### Router

A router a megfelelő komponensek felé irányít a megadott beállítások függvényében, tehát szolgáltatást vagy tartalmat párosít az url-hez. Aki esetleg úgy gondolja, hogy az Apache `mod_rewrite` modulja segítségével egy megfelelően összerakott `.htaccess` fájlban is [éppen ez zajlik](https://medium.com/dotindot/url-routing-megval%C3%B3s%C3%ADt%C3%A1sa-statikus-gener%C3%A1l%C3%A1s%C3%BA-weboldalakon-13ac76bd253c), az jól gondolja.

A `.htaccess` egyszerű és elegáns megoldás, de csak Apache webszervereken működik, ott is csak akkor, ha a `mod_rewrite` modul engedélyezve van az `etc/httpd.conf` konfigurációs állományban. Olyan megoldásra van tehát szükségünk, ami nem csak Apache alatt muzsikál és amennyire csak lehetséges, független a szerver beállításaitól. Erre találták ki az alkalmazásunk részét képező útválasztó (router) mechanizmust és többek közt ezért is irányítottunk az első leckében minden nem létező fájlokra és mappákra irányuló kérelmet az index.php névre hallgató front kontrollerünkre, ahelyett hogy ott helyben (a `.htaccess`-ben) a $_GET tömb egyes elemeihez rendeltük volna az url részeit.

Jelenlegi beállításainkkal bármilyen url-el hívjuk meg az alkalmazásunkat, ugyan azt a választ fogjuk kapni. Ideje ezen változtatnunk. Bemutató alkalmazásunkban a [FastRoute](https://github.com/nikic/FastRoute) komponenst fogjuk használni, de ahogy a többi fejezetben, itt is lecserélhetjük ezt kedvenc csomagunkra, ha van ilyen. Néhány lehetséges alternatíva: [symfony/Routing](https://github.com/symfony/Routing), [Aura.Router](https://github.com/auraphp/Aura.Router), [fuelphp/routing](https://github.com/fuelphp/routing), [Klein](https://github.com/chriso/klein.php).

Telepítsük a kiválasztott routert a már ismert módon, majd írjuk a következő kódot a [`Bootstrap.php`](https://github.com/PatrickLouys/professional-php-sample-code/blob/master/src/Bootstrap.php) fájlba, a `Request` után, de még a `Response` elé:

```php
$dispatcher = \FastRoute\simpleDispatcher(function (\FastRoute\RouteCollector $r) {
    $r->addRoute('GET', '/hello-world', function () {
        echo 'Hello World';
    });
    $r->addRoute('GET', '/another-route', function () {
        echo 'This works too';
    });
});

$routeInfo = $dispatcher->dispatch($request->getMethod(), $request->getPath());
switch ($routeInfo[0]) {
    case \FastRoute\Dispatcher::NOT_FOUND:
        $response->setContent('404 - Page not found');
        $response->setStatusCode(404);
        break;
    case \FastRoute\Dispatcher::METHOD_NOT_ALLOWED:
        $response->setContent('405 - Method not allowed');
        $response->setStatusCode(405);
        break;
    case \FastRoute\Dispatcher::FOUND:
        $handler = $routeInfo[1];
        $vars = $routeInfo[2];
        call_user_func($handler, $vars);
        break;
}
```

A fenti kód első részében az alkalmazásunkban jelenleg elérhető útvonalakat regisztráltuk. Ez után meghívjuk a dispatcher-t ("menetirányító") majd a kapott kimenet függvényében végrehajtódik a switch kifejezés megfelelő része. Ha az útvonal létezik (3. eset), lefut a beállított kezelő-függvény, ha nem létezik (1. eset), vagy nem engedélyezett (2. eset) akkor a megfelelő hibaoldalra irányítjuk a felhasználót.

Ez a megoldás valóban kis alkalmazásoknál megfelelő lehet, de ha további útvonalakat adunk a bootstrap állományunkhoz, hamar áttekinthetetlenné válik. Ennek elkerülésére az útvonalainkat hasznos lehet egy külön fájlban tárolni. Hozzuk létre tehát a `Routes.php` állományt az `src/` mappában, az alábbi tartalommal:

```php
<?php declare(strict_types = 1);

return [
    ['GET', '/hello-world', function () {
        echo 'Hello World';
    }],
    ['GET', '/another-route', function () {
        echo 'This works too';
    }],
];
```

Most írjuk át az útvonal dispatcher részét a `Routes.php` alkalmazásához.

```php
$routeDefinitionCallback = function (\FastRoute\RouteCollector $r) {
    $routes = include('Routes.php');
    foreach ($routes as $route) {
        $r->addRoute($route[0], $route[1], $route[2]);
    }
};

$dispatcher = \FastRoute\simpleDispatcher($routeDefinitionCallback);
```

Ez már egy fokkal jobb, de most az összes kezelő kódunk a `Routes.php`-ban lakozik. Ez nem a legjobb megoldás, ezért a következő fejezetben majd javítani fogjuk.

Ne felejtsük el commitolni munkánkat minden egyes fejezet végén!

[<< előző fejezet](04-http.md) | [következő fejezet >>](06-dispatching-to-a-class.md)
