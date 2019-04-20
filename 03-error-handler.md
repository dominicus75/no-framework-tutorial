[<< előző fejezet](02-composer.md) | [következő fejezet >>](04-http.md)

### Hibakezelő

A hibakezelő segítségével meghatározható, mi történjen akkor, ha a kód futása közben hiba lép fel.

Egy szép hibaoldal, amely sok információt szolgáltat a hibakereséshez, hosszú fejlesztőmunka eredménye. Készülő alkalmazásunk első csomagjának ezt a feladatot kellene ellátnia.

Ha nem szeretnénk most itt leragadni, számos kiváló hibakezelő csomagot találhatunk készen az interneten (github/packagist). Mi most a [filp/whoops](https://github.com/filp/whoops) névre hallgató megoldást fogjuk telepíteni a projektünkhöz, de lehet mást is választani. Ez a keretrendszer nélküli programozás igazi szépsége: teljes ellenőrzés a saját projektünk felett.

Egy másik lehetséges hibakezelő csomag: [PHP-Error](https://github.com/JosephLenton/PHP-Error).

Az új csomag telepítéséhez nyissuk meg a `composer.json` állományunkat, majd adjuk hozzá a `filp/whoops` csomagot a require részhez, az alábbiak szerint:

```php
"require": {
    "php": ">=7.0.0",
    "filp/whoops": "~2.1"
},
```

Ha esetleg saját – vagy más, packagisten nem publikált – hibakezelőt szeretnénk használni, azt is a fentiek szerint tehetjük meg, annyi kiegészítéssel, hogy a Composer figyelmét külön fel kell hívnunk arra, hogy csomagunkat ne az alapértelmezett repositoryban (packagist) keresse, hanem saját tárolónkban (privat repo). Ez esetben a `composer.json` állományban nem elég a `require` alatt megadni, hogy milyen csomagot szeretnénk használni (legyen a csomagunk neve mondjuk `myself/errorhandler`), de a forrást is meg kell jelölnünk, ahol megtalálható (saját gép, Github vagy BitBucket repository). Ez esetben a `composer.json` vonatkozó része valahogy így fog kinézni:

```php
"require": {
    "php": ">=7.0.0",
    "myself/errorhandler": "dev-master"
},
"repositories": [
    {
        "type": "git",
        "url": "/path/to/errorhandler/.git"
    }
],
```

Ha GitHub vagy BitBucket tárolót használunk, akkor az `url` értéke értelemszerűen `https://github.com/myself/errorhandler.git` lesz. Saját csomag használata esetén figyeljünk arra, hogy – amennyiben közzé akarjuk tenni a művünket, illetve éles környezetben használni – akkor mindenképpen nyilvános tárolót (GitHub, BitBucket, stb.) adjunk meg. Azért ne a gépünkön található `.git` állományra hivatkozzunk a `repositories` tömbben, mert ahhoz csak mi férünk hozzá, így ha valaki telepíti a programunkat, a teljesíthetetlen függőség(ek) miatt nem tudja használni. Ha csak mi használjuk a projektünket (mint jelen esetben), akkor nyugodtan hivatkozhatunk a gépünkön lakó saját tárolónkra is.

Ha ezzel kész vagyunk, akkor futtassuk a `composer update` parancsot konzolban, hogy a Composer telepíthesse az új csomagot és frissítse a `composer.lock` állományt.

A vágyott hibakezelő immár a `vendor` mappánkban lakik, használni viszont még nem tudjuk. A PHP ugyanis nem tudja, hol találja meg a kért osztályokhoz tartozó forrásállományokat. Ennek áthidalásához szükségünk lesz egy – ideális esetben a [PSR-4](http://www.php-fig.org/psr/psr-4/) szabványt megvalósító – autoloaderre. A Composer egész véletlenül éppen rendelkezik egy ilyennel. Ahhoz, hogy használni tudjuk, írjuk ezt (`require __DIR__ . '/../vendor/autoload.php';`) a `Bootstrap.php` állományunkba, majd adjuk ki konzolban a `composer dump-autoload` parancsot, hogy a Composer frissen telepített csomagunkat is felvegye az autoloaderbe (a `dump-autoload`-ra minden új csomag telepítése után szükség van).

Fejlesztéshez, teszteléshez tökéletesen megfelel az így kapott psr-4 autoloader, viszont éles környezetben lassúnak bizonyulhat. Ezért élesítés/publikáció előtt (amikor az aktív fejlesztést már befejeztük), nem árt optimalizálni autoloaderünket a `composer dump-autoload --optimize` parancs segítségével. Az `--optimize` kapcsoló hatására psr-4 autoloaderünkből a Composer egy osztálytérképet (`vendor/composer/autoload_classmap.php`) generál, minden egyes osztályhoz külön hozzárendelve a forrásfájl elérési útját, némiképp lerövidítve a kért osztályok keresését.

**Fontos:** Soha ne engedjük, hogy programunk éles környezetben a felhasználók által látható hibákat dobáljon. Ezek a – fejlesztés közben számunkra nagyon is hasznos – hibaüzenetek segítséget nyújthatnak mindenkinek, aki sebezhetőségeket keresve hozzá szeretne férni a rendszerünkhöz. Hiba esetén a program mindig egy felhasználóknak szánt hibaoldalt mutasson, ahol éppen annyi információt tegyünk közzé, amennyi a felhasználóra tartozik (pl. egy 500-as HTTP hibaoldal), majd a hiba lényegi részét küldessük el magunknak géplevélben, esetleg naplózzuk (errorlog). Ügyeljünk arra, hogy a telepített/élesített környezet esetleges hibáit csak mi láthassuk.

Bár a fejlesztés szempontjából nem sok értelme van, de mégis szeretnénk magunknak egy szép hibaoldalt, akkor helyezzünk el egy környezetválasztó kapcsolót a kódunkban, és állítsuk be `development` értékre.

A hibakezelőnk regisztrációját követően dobjunk egy `Exception`-t, kipróbálandó, hogy minden megfelelően működik-e. A `Bootstrap.php` állományunk vonatkozó része ennek megfelelően így fog kinézni:

```php
<?php declare(strict_types = 1);

namespace Example;

require __DIR__ . '/../vendor/autoload.php';

error_reporting(E_ALL);

$environment = 'development';

/**
* Hibakezelő regisztrálása
*/
$whoops = new \Whoops\Run;
if ($environment !== 'production') {
    $whoops->pushHandler(new \Whoops\Handler\PrettyPageHandler);
} else {
    $whoops->pushHandler(function($e){
        echo 'Todo: Friendly error page and send an email to the developer';
    });
}
$whoops->register();

throw new \Exception;

```

Ideális esetben egy hiba oldalt kellene látnunk, azon sor kiemelésével, ahol az `Exception` eldobásra került, illetve a hiba felmerült. Ha mégsem, akkor menjünk vissza a forrásba bogarászni (hibát keresni), míg a várt kimenetet nem kapjuk. Ennek végeztével ne felejtsünk el commitolni (git).

[<< előző fejezet](02-composer.md) | [következő fejezet >>](04-http.md)
