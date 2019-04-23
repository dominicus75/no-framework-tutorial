[<< előző fejezet](05-router.md) | [következő fejezet >>](07-inversion-of-control.md)

### Átirányítás egy osztályba

A jelen útmutató nyomán szárba szökkenő bemutató alkalmazásunk nem valósítja meg az [MVC (Model-View-Controller)](https://hu.wikipedia.org/wiki/Modell-n%C3%A9zet-vez%C3%A9rl%C5%91) tervezési mintát. Ennek az az oka, hogy szerintem az MVC-t nem lehet megfelelően implementálni a PHP-ben, legalábbis nem az eredetileg kigondolt módon. A témával bővebben az [MVC-útmutató kezdőknek](http://blog.ircmaxell.com/2014/11/a-beginners-guide-to-mvc-for-web.html) című írás foglalkozik (angolul).
*A fordító megjegyzése: magyarul nem találtam a szerző által hivatkozott cikkel azonos tartalmút, csak [Pásztor János](https://www.refaktor.hu/tiszta-kod-6-resz-beszelnunk-kell-az-mvc-rol/) vagy [Szél Péter](https://kszk.sch.bme.hu/2012/03/18/mi-van-az-mvc-tervezesi-mintan-tul/) írása tér ki az MVC webes alkalmazásának korlátaira. Sallai András pedig így [foglalja össze](https://szit.hu/doku.php?id=oktatas:programoz%C3%A1s:mvc) a Patrick Louys által felvetett problémát: "A web erősen támaszkodik a HTTP protokollra, amely állapotmentes. Ez azt jelenti, hogy nincs folytonos kapcsolat a böngésző és a webszerver között. Minden kérés egy újabb kapcsolatot hoz létre. Ha böngésző megkapta a választ, zárja a kapcsolatot. Ez egy olyan helyzet, amelyre nem gondoltak az eredeti MVC fejlesztői."*.

Tehát egyelőre felejtsük el az MVC-t és helyette inkább aggódjunk a [vonatkozások szétválasztása (SoC)](http://www.clean-code-developer.hu/separation-of-concerns-soc/) miatt...

Szükségünk lesz egy kifejező névre a kérelmeket kezelő osztályaink számára. Ehhez az útmutatóhoz a `Controllers` elnevezést fogjuk használni, mert ez ismerős lehet a keretrendszereket használók számára is. De akár a `Handlers` nevet is adhatjuk nekik, ízlés dolga.

Hozzunk létre az `src/` mappában egy `Controllers` alkönyvtárat. Itt fogjuk elhelyezni az összes vezérlő (controller) osztályunkat, elsőként a `Homepage.php` állományt.

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

class Homepage
{
    public function show()
    {
        echo 'Hello World';
    }
}
```

The autoloader will only work if the namespace of a class matches the file path and the file name equals the class name. At the beginning I defined `Example` as the root namespace of the application so this is referring to the `src/` folder.

Now let's change the hello world route so that it calls your new class method instead of the closure. Change your `Routes.php` to this:

```php
return [
    ['GET', '/', ['Example\Controllers\Homepage', 'show']],
];
```

Instead of just a callable you are now passing an array. The first value is the fully namespaced classname, the second one the method name that you want to call.

To make this work, you will also have to do a small refactor to the routing part of the `Bootstrap.php`:

```php
case \FastRoute\Dispatcher::FOUND:
    $className = $routeInfo[1][0];
    $method = $routeInfo[1][1];
    $vars = $routeInfo[2];

    $class = new $className;
    $class->$method($vars);
    break;
```

So instead of just calling a method you are now instantiating an object and then calling the method on it.

Now if you visit `http://localhost:8000/` everything should work. If not, go back and debug. And of course don't forget to commit your changes.

[<< előző fejezet](05-router.md) | [következő fejezet >>](07-inversion-of-control.md)
