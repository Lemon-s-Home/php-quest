# 14. Composer берёт загрузку на себя

## Мы написали autoload руками

В прошлом уроке мы сделали правило:

```text
Game\Weapon\Sword → src/Weapon/Sword.php
```

Работает. Но это правило уже давно стандартизировано. В PHP для этого обычно используют Composer.

Composer умеет две важные вещи:

- подключать твои классы по namespace;
- ставить чужие библиотеки.

Начнём с первого.

## composer.json

В корне проекта создай файл `composer.json`:

```json
{
    "autoload": {
        "psr-4": {
            "Game\\": "src/"
        }
    }
}
```

Это значит:

```text
классы с namespace Game\ лежат в папке src/
```

Теперь выполни:

```bash
composer dump-autoload
```

Composer создаст папку `vendor/` и файл:

```text
vendor/autoload.php
```

## main.php

Свой `autoload.php` больше не нужен.

```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use Game\Fighter\Warrior;
use Game\Weapon\Sword;

$hero = new Warrior('Knight', 100, new Sword('Sword', 25));

echo $hero->describe() . "\n";
```

Одна строка подключает всё:

```php
require_once __DIR__ . '/vendor/autoload.php';
```

## PSR-4

Правило, которое использует Composer, называется PSR-4.

Главная идея простая:

```text
Game\Weapon\Sword
```

совпадает с:

```text
src/Weapon/Sword.php
```

Если класс называется `Game\Fighter\Warrior`, файл должен лежать здесь:

```text
src/Fighter/Warrior.php
```

Если папка или имя файла не совпадают, Composer не найдёт класс.

## Первая dev-библиотека

Для отладки удобно смотреть, что лежит внутри объекта.

Обычный `var_dump()` шумный:

```php
var_dump($hero);
```

Поставим нормальный инструмент:

```bash
composer require --dev symfony/var-dumper
```

`--dev` значит: эта библиотека нужна разработчику, а не самой игре.

После установки появляется функция `dump()`:

```php
dump($hero);
```

И функция `dd()`:

```php
dd($hero);
```

`dd` означает dump and die: показать значение и остановить программу.

## Отладка объекта

```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use Game\Fighter\Warrior;
use Game\Weapon\Sword;

$hero = new Warrior('Knight', 100, new Sword('Sword', 25));

dump($hero);

echo $hero->describe() . "\n";
```

Теперь можно увидеть не только строку описания, но и сам объект: его поля, вложенное оружие, текущее состояние.

Это особенно полезно в игре, где объект может держать внутри другие объекты.

## `require` и `require-dev`

В `composer.json` теперь может быть две секции:

```json
{
    "autoload": {
        "psr-4": {
            "Game\\": "src/"
        }
    },
    "require-dev": {
        "symfony/var-dumper": "^7.0"
    }
}
```

Номер версии может отличаться. Composer подберёт подходящую версию сам.

`require` — нужно программе.

`require-dev` — нужно тебе во время разработки.

VarDumper не часть игры. Игроку не нужно знать, как ты смотришь объект изнутри.

## Шпаргалка

- `composer.json` описывает проект
- `autoload.psr-4` связывает namespace и папку
- `composer dump-autoload` пересобирает автозагрузку
- `vendor/autoload.php` подключают один раз
- `composer require --dev symfony/var-dumper` ставит инструмент для отладки
- `dump($object)` показывает объект
- `dd($object)` показывает объект и останавливает программу

## Напоследок

1. `vendor/` обычно не пишут руками. Это папка Composer. Он сам кладёт туда установленные библиотеки и свой autoload.

2. `composer.lock` запоминает точные версии библиотек. Это нужно, чтобы у тебя и у другого человека проект запускался с одинаковыми версиями.

3. `dump()` — глобальная функция. Это удобно для отладки, но не стоит строить вокруг неё архитектуру игры. Посмотрел объект, понял проблему, убрал `dump`.

## Практика

### 14.1. Перейди на Composer autoload

Возьми проект с `src/` и namespace `Game\`.

Создай `composer.json` с PSR-4:

```json
"Game\\": "src/"
```

Выполни:

```bash
composer dump-autoload
```

Удали подключение своего `autoload.php`. В `main.php` должен остаться только:

```php
require_once __DIR__ . '/vendor/autoload.php';
```

### 14.2. Посмотри объект изнутри

Поставь VarDumper:

```bash
composer require --dev symfony/var-dumper
```

Создай героя с оружием и выведи:

```php
dump($hero);
```

Потом нанеси герою урон и снова вызови `dump($hero)`.

Проверь, что в дампе видно изменение HP.
