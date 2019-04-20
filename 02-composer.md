[<< előző fejezet](01-front-controller.md) | [következő fejezet >>](03-error-handler.md)

### Composer

A [Composer](https://getcomposer.org/) a PHP függőségkezelője.

Az a tény, hogy nem használunk keretrendszert, nem jelenti azt, hogy minden egyes alkalommal újra fel kell találnunk a kereket, akárhányszor új projektbe kezdünk. A Composer segítségével könnyedén telepíthetünk alkalmazásunkhoz akár harmadik féltől származó csomagokat (library-kat vagy osztályokat), vagy saját korábbi csomagjainkat (lokális vagy távoli privát tárolókból).

Ha még nincs telepítve gépünkön a Composer, itt az ideje, hogy orvosoljuk ezt a hiányosságot. A program a legtöbb Linux disztribúció tárolóiban megtalálható, így nincs más teendőnk, mint kiadni az `apt-get install Composer` parancsot (Debian alapú rendszeren). A projektünkhöz esetleg szükséges Composer csomagokat megtalálhatjuk a [Packagisten](https://packagist.org/), ami a Composer alapértelmezett csomagtárolója.

Ha már rendelkezésre áll a Composer, hozzunk létre projektünk gyökérkönyvtárában egy `composer.json` állományt (ha ebből a könyvtárból konzolban kiadjuk a `composer init` parancsot, akkor a program megteszi ezt helyettünk, egyenként bekérve a szükséges adatokat). A Composer ebből az állományból nyeri projektünk és függőségeinek konfigurációs adatait. Ügyeljünk arra, hogy a fájlt érvényes JSON formátumban készítsük el, különben a Composer hibát fog dobni.

Írjuk az alábbiakat az imént létrehozott állományba:

```json
{
    "name": "Project name",
    "description": "Your project description",
    "keywords": ["Your keyword", "Another keyword"],
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your@email.com",
            "role": "Creator / Main Developer"
        }
    ],
    "require": {
        "php": ">=7.0.0"
    },
    "autoload": {
        "psr-4": {
            "Example\\": "src/"
        }
    }
}
```

Az autoload tömbből kiderül, hogy jelen projektünkhöz az `Example` névteret használjuk. A jelen útmutató összes példájában ezt fogjuk alkalmazni. Ha más névteret vagy névtereket szeretnénk beállítani, akkor ezt kell lecserélni, vagy az `autoload/psr-4` tömböt értelemszerűen bővíteni.

Nyissunk meg egy új konzolablakot, majd navigáljunk a projekt gyökér könyvtárába. Itt adjuk ki a `composer update` parancsot.

A parancs hatására a Composer létrehozza projektünk gyökerében a `composer.lock` állományt, amely zárolja a függőségeket és a vendor mappát.

A `composer.lock` fájl verziókövetése erősen ajánlott, mert a Composer ebben tárolja a letöltött függőségek konkrét verziószámát. Ha ez rendelkezésre áll, akkor a Composer ez, nem pedig a `composer.json` alapján fogja letölteni a függőségeket. Ez akkor fontos igazán, ha egy projekten többen is dolgoznak. A `composer.lock` biztosítja, hogy mindenkinél azonos verziószámú függőségek legyenek telepítve, megkönnyítve a folyamatos integrációs tesztek (mint a [Travis CI](https://travis-ci.org/)) futtatását is.

Ez magával vonja azt is, hogy [nem kell a függőségek tényleges forrását a git tárolónkba helyezni](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md), hiszen a Composer automatikusan letölti azokat, ha valaki telepíti a projektünket. Éppen ezért a `.gitignore` állományba vegyük fel a függőségeinket tartalmazó vendor mappát is:

```
vendor/
```

#### Ajánlott irodalom:

Botond (Linuxvilág): [Composer PHP csomagkezelő telepítése](https://www.linuxportal.info/leirasok/web-hoszting/egyeb/composer-php-csomagkezelo-telepitese)

Papp Krisztián: [A PHP-fejlesztők kedvenc zeneszerzője](https://www.letscode.hu/2015/03/12/composer-a-php-fejlesztok-kedvenc-zeneszerzoje)

Sallai András: [Composer](https://szit.hu/doku.php?id=oktatas:web:composer)


[<< előző fejezet](01-front-controller.md) | [következő fejezet >>](03-error-handler.md)
