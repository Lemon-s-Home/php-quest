# 13. Классы подключаются сами

## Слишком много `require_once`

После namespace код стал чище, но `main.php` всё ещё выглядит так:

```php
require_once __DIR__ . '/src/Weapon/Weapon.php';
require_once __DIR__ . '/src/Weapon/Sword.php';
require_once __DIR__ . '/src/Weapon/Axe.php';
require_once __DIR__ . '/src/Fighter/Warrior.php';
require_once __DIR__ . '/src/Fighter/Monster.php';
require_once __DIR__ . '/src/Effect/Poisoned.php';
```

Добавил класс — добавь `require_once`. Забыл — программа падает.

Хочется один раз объяснить PHP правило:

```text
Game\Weapon\Sword → src/Weapon/Sword.php
```

И больше не писать список файлов руками.

## Autoload

PHP умеет вызывать функцию, когда встречает неизвестный класс.

```php
spl_autoload_register(function (string $class): void {
    echo "PHP ищет класс: $class\n";
});
```

Если после этого написать:

```php
new Game\Weapon\Sword('Sword', 25);
```

PHP передаст в autoload строку:

```text
Game\Weapon\Sword
```

Наша задача — превратить эту строку в путь к файлу.

## autoload.php

Файл `autoload.php`:

```php
<?php

declare(strict_types=1);

spl_autoload_register(function (string $class): void {
    $prefix = 'Game\\';
    $baseDir = __DIR__ . '/src/';

    if (!str_starts_with($class, $prefix)) {
        return;
    }

    $relativeClass = substr($class, strlen($prefix));
    $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';

    if (file_exists($file)) {
        require_once $file;
    }
});
```

Разберём путь:

```text
Game\Weapon\Sword
```

Убираем начало `Game\`:

```text
Weapon\Sword
```

Меняем `\` на `/`:

```text
Weapon/Sword
```

Добавляем `src/` и `.php`:

```text
src/Weapon/Sword.php
```

## main.php

Теперь `main.php` подключает только autoload:

```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/autoload.php';

use Game\Fighter\Warrior;
use Game\Weapon\Sword;

$hero = new Warrior('Knight', 100, new Sword('Sword', 25));

echo $hero->describe() . "\n";
```

`main.php` больше не знает список файлов. Он знает только классы.

## Почему это ООП

Autoload не делает объекты умнее. Но он помогает проекту расти.

Когда игра становится больше, появляются папки:

```text
src/
├── Fighter/
├── Weapon/
├── Effect/
├── Attack/
├── Terminal/
└── Game.php
```

Если каждый файл подключать руками, ты будешь думать не об игре, а о списке `require_once`.

Autoload убирает шум.

## Шпаргалка

- `spl_autoload_register(...)` регистрирует функцию автозагрузки
- PHP вызывает autoload, когда встречает неизвестный класс
- autoload получает полное имя класса строкой
- `Game\Weapon\Sword` можно превратить в `src/Weapon/Sword.php`
- `main.php` подключает только `autoload.php`
- namespace и autoload — разные вещи, но они хорошо работают вместе

## Напоследок

1. PHP может зарегистрировать несколько autoload-функций. Если первая не нашла класс, PHP попробует следующую.

2. `spl` в `spl_autoload_register` означает Standard PHP Library. Это старое название набора встроенных инструментов PHP.

3. В реальных проектах такой autoload почти всегда пишет не человек, а Composer. Но один раз написать его руками полезно: после этого Composer перестаёт казаться магией.

## Практика

### 13.1. Убери ручные require

Возьми проект из прошлого урока.

Создай `autoload.php`, который загружает классы из namespace `Game\` из папки `src/`.

В `main.php` должен остаться только один `require_once`:

```php
require_once __DIR__ . '/autoload.php';
```

Проверь, что бойцы и оружие всё ещё создаются.

### 13.2. Новый класс без нового require

Добавь класс `Game\Effect\Poisoned` в файл:

```text
src/Effect/Poisoned.php
```

В `main.php` используй его через `use Game\Effect\Poisoned;`.

Если autoload написан правильно, новый `require_once` добавлять не придётся.
