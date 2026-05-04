# 17. Числа уезжают в конфиг

## Код и данные

Сейчас оружие создаётся прямо в коде:

```php
$weapon = new Frost(
    new BasicWeapon('Ice Sword', 20),
    7,
);
```

Для урока это нормально. Видно, как декоратор оборачивает оружие.

Но числа `20` и `7` — это не поведение. Это настройки игры.

Если ледяной бонус нужно поменять с `7` на `9`, не хочется искать это число по коду. Лучше вынести такие числа в конфиг.

Оружие пока собирается руками:

```php
new Frost(new BasicWeapon(...), ...);
```

Меняется только одно: числа лежат в PHP-файле с настройками.

## Что где хранить

В коде:

- `BasicWeapon` знает, как наносить базовый урон;
- `Frost` знает, как добавить ледяной урон;
- `Fire` знает, как добавить огненный урон;
- `main.php` пока сам решает, какие объекты соединить.

В конфиге:

- базовый урон оружия;
- бонусный урон эффектов;
- другие числа баланса, если они появятся позже.

В сохранении игры позже будет другое:

- какое оружие сейчас у героя;
- сколько HP осталось;
- какие временные эффекты активны;
- где находится игрок.

Не смешивай это. Конфиг описывает настройки мира. Сохранение описывает конкретную партию.

## config/weapons.php

Создай файл:

```text
config/weapons.php
```

```php
<?php

declare(strict_types=1);

return [
    'ice_sword' => [
        'damage' => 20,
    ],
];
```

Здесь нет класса `BasicWeapon` и нет `new`.

Это просто числа.

## config/effects.php

Создай ещё один файл:

```text
config/effects.php
```

```php
<?php

declare(strict_types=1);

return [
    'frost' => [
        'bonusDamage' => 7,
    ],
];
```

Здесь тоже нет класса `Frost`.

Ключ `frost` — это адрес настроек для ледяного эффекта.

Загрузить конфиги можно обычным `require`:

```php
$weapons = require __DIR__ . '/config/weapons.php';
$effects = require __DIR__ . '/config/effects.php';
```

## Базовое оружие

```php
<?php

declare(strict_types=1);

namespace Game\Weapon;

class BasicWeapon implements Weapon
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

`BasicWeapon` не знает, откуда пришёл урон. Из конфига, из теста, из кода — ему всё равно.

## Декоратор Frost

```php
<?php

declare(strict_types=1);

namespace Game\Weapon;

class Frost implements Weapon
{
    private int $bonusDamage;

    public function __construct(
        private Weapon $base,
        array $effects,
    ) {
        $this->bonusDamage = $effects['frost']['bonusDamage'];
    }

    public function strike(): int
    {
        return $this->base->strike() + $this->bonusDamage;
    }

    public function describe(): string
    {
        return $this->base->describe() . ' + лёд (' . $this->bonusDamage . ')';
    }
}
```

`Frost` сам знает, что его настройки лежат под ключом `frost`.

В коде нет сравнения:

```php
if ($type === 'frost')
```

И нет константы ради константы.

Есть обычный объект, который при рождении забирает своё число из конфига.

После конструктора внутри `Frost` лежит уже простое число `$bonusDamage`.

## main.php

```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use Game\Weapon\BasicWeapon;
use Game\Weapon\Frost;

$weapons = require __DIR__ . '/config/weapons.php';
$effects = require __DIR__ . '/config/effects.php';

$weapon = new Frost(
    new BasicWeapon('Ice Sword', $weapons['ice_sword']['damage']),
    $effects,
);

echo $weapon->describe() . "\n";
echo 'Урон: ' . $weapon->strike() . "\n";
```

`main.php` всё ещё собирает объект руками.

Это нормально для этого урока.

Цель сейчас не автоматическая сборка оружия. Цель — убрать числа из кода.

## Шпаргалка

- PHP-конфиг может просто `return [...]`
- конфиг хранит числа баланса
- объект может получить конфиг в конструкторе
- объект сам знает, где лежат его настройки
- `main.php` пока сам собирает объекты
- сохранение игры позже будет хранить состояние партии, а не PHP-объекты

## Напоследок

1. В играх часто меняют числа баланса отдельно от поведения: урон, здоровье, цену предметов, шанс выпадения.

2. В больших играх такие числа часто редактируют не программисты, а геймдизайнеры через таблицы или специальные редакторы.

3. Иногда изменение одного числа полностью ломает баланс игры. Поэтому конфиг — это не мусорный ящик, а важная часть проекта.

## Практика

### 17.1. Урон из конфига

Создай `config/weapons.php`:

```php
<?php

declare(strict_types=1);

return [
    'training_sword' => [
        'damage' => 10,
    ],
    'heavy_axe' => [
        'damage' => 26,
    ],
];
```

Создай два `BasicWeapon`.

Названия можно оставить в коде, но урон бери из конфига.

Выведи описание и урон каждого оружия.

### 17.2. Огонь из конфига

Добавь декоратор `Fire`.

Он должен добавлять урон и менять описание.

В `config/effects.php` добавь настройки:

```php
<?php

declare(strict_types=1);

return [
    'fire' => [
        'bonusDamage' => 9,
    ],
];
```

`Fire` должен сам взять число из `$effects['fire']['bonusDamage']`.

Создай огненный топор:

```php
$weapon = new Fire(
    new BasicWeapon('Fire Axe', $weapons['heavy_axe']['damage']),
    $effects,
);
```

Проверь, что в `main.php` нет числа `9`.
