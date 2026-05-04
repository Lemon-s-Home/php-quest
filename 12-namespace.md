# 12. Классы получают адрес

## Проблема: имена заканчиваются

Ты уже начал раскладывать код по папкам:

```text
Weapon/
├── Weapon.php
└── Sword.php

Fighter/
├── Fighter.php
└── Warrior.php
```

Это правильное движение. Когда классов становится много, держать их в одном файле больно.

Но папка сама по себе для PHP почти ничего не значит. Если в двух местах появится класс `Sword`, PHP увидит просто два класса с одним именем:

```php
class Sword
{
}
```

И скажет: «Такой класс уже есть».

Нужен не просто файл. Нужен полный адрес класса.

## Namespace

`namespace` добавляет классу адрес:

```php
<?php

declare(strict_types=1);

namespace Game\Weapon;

class Sword
{
}
```

Теперь это не просто `Sword`. Полное имя класса:

```php
Game\Weapon\Sword
```

Можно сделать другой `Sword`:

```php
<?php

declare(strict_types=1);

namespace Game\Loot;

class Sword
{
}
```

Это уже другой класс:

```php
Game\Loot\Sword
```

Одинаковое короткое имя. Разный адрес.

## Папка и namespace

Хорошее правило:

```text
src/Weapon/Sword.php  →  Game\Weapon\Sword
src/Fighter/Warrior.php → Game\Fighter\Warrior
```

Папка помогает человеку найти файл. `namespace` помогает PHP отличить один класс от другого.

Пока PHP сам не умеет находить файл по namespace. Это будет в следующем уроке. Сейчас мы всё ещё используем `require_once`.

## Interface тоже получает namespace

Файл `src/Weapon/Weapon.php`:

```php
<?php

declare(strict_types=1);

namespace Game\Weapon;

interface Weapon
{
    public function strike(): int;
    public function describe(): string;
}
```

Файл `src/Weapon/Sword.php`:

```php
<?php

declare(strict_types=1);

namespace Game\Weapon;

class Sword implements Weapon
{
    public function __construct(
        private string $name,
        private int $damage,
    ) {
    }

    public function strike(): int
    {
        return $this->damage;
    }

    public function describe(): string
    {
        return $this->name . ' (урон: ' . $this->damage . ')';
    }
}
```

`Sword` видит `Weapon`, потому что они в одном namespace: `Game\Weapon`.

## `use`

Теперь воин находится в другом namespace.

Файл `src/Fighter/Warrior.php`:

```php
<?php

declare(strict_types=1);

namespace Game\Fighter;

use Game\Weapon\Weapon;

class Warrior
{
    public function __construct(
        private string $name,
        private int $hp,
        private Weapon $weapon,
    ) {
    }

    public function attack(Warrior $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }

    public function takeDamage(int $damage): void
    {
        $this->hp = max(0, $this->hp - $damage);
    }

    public function describe(): string
    {
        return $this->name . ': ' . $this->hp . ' HP, ' . $this->weapon->describe();
    }
}
```

Строка:

```php
use Game\Weapon\Weapon;
```

говорит: когда я пишу `Weapon`, я имею в виду `Game\Weapon\Weapon`.

Без `use` пришлось бы писать полное имя:

```php
private \Game\Weapon\Weapon $weapon
```

Работает, но читать хуже.

## main.php

Пока подключаем файлы руками:

```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/src/Weapon/Weapon.php';
require_once __DIR__ . '/src/Weapon/Sword.php';
require_once __DIR__ . '/src/Fighter/Warrior.php';

use Game\Fighter\Warrior;
use Game\Weapon\Sword;

$hero = new Warrior('Knight', 100, new Sword('Sword', 25));
$enemy = new Warrior('Orc', 80, new Sword('Axe', 20));

$hero->attack($enemy);

echo $hero->describe() . "\n";
echo $enemy->describe() . "\n";
```

`require_once` подключает файлы. `use` сокращает длинные имена классов. Это разные вещи.

## Шпаргалка

- `namespace Game\Weapon;` даёт классу полный адрес
- `Game\Weapon\Sword` и `Game\Loot\Sword` — разные классы
- `use Game\Weapon\Sword;` позволяет писать коротко: `new Sword(...)`
- папка не подключает файл сама
- `require_once` пока всё ещё нужен
- namespace должен стоять в начале файла, после `declare(strict_types=1);`

## Напоследок

1. В C++ похожую роль играет `namespace`, но файлы подключаются через `#include`. В PHP файл подключается во время выполнения программы.

2. Обратный слеш `\` в имени класса выглядит странно, потому что обычный слеш `/` уже занят путями файлов. Поэтому путь файла и имя класса похожи, но не одинаковы.

3. В больших PHP-проектах почти никогда не пишут классы без namespace. Класс без namespace живёт в глобальной куче имён, а там быстро становится тесно.

## Практика

### 12.1. Оружие получает адрес

Создай структуру:

```text
src/
└── Weapon/
    ├── Weapon.php
    ├── Sword.php
    └── Axe.php
```

Все классы и интерфейсы должны жить в namespace `Game\Weapon`.

В `main.php` подключи файлы через `require_once`, создай меч и топор, выведи их описание.

### 12.2. Боец использует оружие

Создай структуру:

```text
src/
├── Fighter/
│   └── Warrior.php
└── Weapon/
    ├── Weapon.php
    ├── Sword.php
    └── Axe.php
```

`Warrior` должен жить в namespace `Game\Fighter` и принимать `Game\Weapon\Weapon` в конструкторе.

В `main.php` создай двух бойцов с разным оружием. Проведи один удар и выведи состояние обоих.
