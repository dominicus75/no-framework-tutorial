[<< előző fejezet](05-router.md) | [következő fejezet >>](07-inversion-of-control.md)

### Átirányítás egy osztályba

A jelen útmutató nyomán szárba szökkenő bemutató alkalmazásunk nem valósítja meg az [MVC (Model-View-Controller)](https://hu.wikipedia.org/wiki/Modell-n%C3%A9zet-vez%C3%A9rl%C5%91) tervezési mintát. Ennek az az oka, hogy szerintem az MVC-t nem lehet megfelelően implementálni a PHP-ben, legalábbis nem az eredetileg kigondolt módon. A témával bővebben az [MVC-útmutató kezdőknek](http://blog.ircmaxell.com/2014/11/a-beginners-guide-to-mvc-for-web.html) című írás foglalkozik (angolul).

*A fordító megjegyzése: magyarul nem találtam a szerző által hivatkozott cikkel azonos tartalmút, csak [Pásztor János](https://www.refaktor.hu/tiszta-kod-6-resz-beszelnunk-kell-az-mvc-rol/) vagy [Szél Péter](https://kszk.sch.bme.hu/2012/03/18/mi-van-az-mvc-tervezesi-mintan-tul/) írása tér ki az MVC webes alkalmazásának korlátaira. Sallai András pedig így [foglalja össze](https://szit.hu/doku.php?id=oktatas:programoz%C3%A1s:mvc) a Patrick Louys által felvetett problémát: **"A web erősen támaszkodik a HTTP protokollra, amely állapotmentes. Ez azt jelenti, hogy nincs folytonos kapcsolat a böngésző és a webszerver között. Minden kérés egy újabb kapcsolatot hoz létre. Ha böngésző megkapta a választ, zárja a kapcsolatot. Ez egy olyan helyzet, amelyre nem gondoltak az eredeti MVC fejlesztői.**"*.

Tehát egyelőre felejtsük el az MVC-t és helyette inkább aggódjunk a [vonatkozások szétválasztása (SoC)](http://www.clean-code-developer.hu/separation-of-concerns-soc/) miatt...

Szükségünk lesz egy kifejező névre a kérelmeket kezelő osztályaink számára. Ehhez az útmutatóhoz a `Controllers` elnevezést fogjuk használni, mert ez ismerős lehet a keretrendszereket használók számára is. De akár a `Handlers` nevet is adhatjuk neki, ízlés dolga.

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

Autoloaderünk csak akkor fog megfelelően működni, ha osztályunk [névtere](https://www.letscode.hu/2015/02/24/de-jegyezd-meg-jol-mig-a-fold-kerek-mindig-lesznek-nevterek) megegyezik a tartalmazó könyvtár elérési útjával, illetve az állománynév az osztály nevével. Vagyis: a könyvtárak jelképezik a névtereket, míg a bennük lévő állományok az osztályokat. A [második fejezetben](02-composer.md) a `composer.json` "autoload" szekciójában ezért alkalmazásunk `Example` nevű gyökérnévtere az `src/` mappára mutat, tehát a Composer autoloadere innen indul a betöltendő osztály keresésére, így az imént létrehozott `Example\Controllers\Homepage` osztályt is az `src/Controllers/Homepage.php` állományban fogja keresgélni.

Ezért most változtassuk meg a `Routes.php` állományban az [előző fejezetben](05-router.md) létrehozott tömbünket az alábbiak szerint:

```php
return [
    ['GET', '/', ['Example\Controllers\Homepage', 'show']],
];
```

Az előző fejezetben a visszatérési értékként megadott tömb harmadik elemeként létrehozott névtelen függvény helyett most ugyanott egy tömböt fogunk vissza adni. A tömb kulcsa a teljes osztálynév (`Example\Controllers\Homepage`), amelyhez a hivatkozott osztály azon metódusát rendeljük értékként (`show`), amelyet szeretnénk meghívni akkor, ha `GET` metódussal (első tömbelem), a nyitólapra (`/` - második tömbelem) irányuló kérelem érkezik.

Ahhoz, hogy változtatásaink valóban működjenek, némi módosításra lesz szükségünk a `Bootstrap.php` routing részében is:

```php
case \FastRoute\Dispatcher::FOUND:
    $className = $routeInfo[1][0];
    $method = $routeInfo[1][1];
    $vars = $routeInfo[2];

    $class = new $className;
    $class->$method($vars);
    break;
```

Tehát ahelyett, hogy egy anonim függvényt hívnánk mint eddig, most létrehozunk egy objektumot majd meghívjuk annak egyik metódusát.

Látogassuk meg a `http://localhost:8000/` címet és győződjünk meg róla, hogy minden működik. Ha nem, irány bogarászni... Ha készen vagyunk, természetesen ne felejtsük el commitolni a változtatásainkat.

[<< előző fejezet](05-router.md) | [következő fejezet >>](07-inversion-of-control.md)
