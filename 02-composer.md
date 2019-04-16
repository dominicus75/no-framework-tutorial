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

In the autoload part you can see that I am using the `Example` namespace for the project. You can use whatever fits your project there, but from now on I will always use the `Example` namespace in my examples. Just replace it with your namespace in your own code.

Open a new console window and navigate into your project root folder. There run `composer update`.

Composer creates a `composer.lock` file that locks in your dependencies and a vendor directory.

Committing the `composer.lock` file into version control is generally good practice for projects. It allows continuation testing tools (such as [Travis CI](https://travis-ci.org/)) to run the tests against the exact same versions of libraries that you're developing against. It also allows all people who are working on the project to use the exact same version of libraries i.e. it eliminates a source of "works on my machine" problems.

That being said, [you don't want to put the actual source code of your dependencies in your git repository](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md). So let's add a rule to our `.gitignore` file:

```
vendor/
```

Now you have successfully created an empty playground which you can use to set up your project.

[<< előző fejezet](01-front-controller.md) | [következő fejezet >>](03-error-handler.md)
