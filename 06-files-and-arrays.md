# 06. Один класс — один файл

В прошлых уроках все классы жили в одном файле. Пять классов — уже неудобно листать. Десять — невозможно.

## C++ → PHP: в чём разница

В C++ класс делился на два файла: `.h` (объявление) и `.cpp` (реализация). Это нужно было компилятору — он должен знать, что класс существует, прежде чем его использовать.

В PHP такого разделения нет. Один класс — один `.php` файл. Объявление и реализация вместе.

## `require_once`

Чтобы использовать класс из другого файла — подключи его:

```php
require_once 'Weapon.php';
require_once 'Armor.php';
require_once 'Player.php';
```

`require_once` читает файл и выполняет его. Если подключить один файл дважды — выполнится только один раз. Если файл не найден — ошибка и скрипт останавливается.

## Структура проекта

```
06-project/
├── Weapon.php
├── Armor.php
├── Player.php
├── Monster.php
└── main.php
```

**Weapon.php:**

```php
<?php

declare(strict_types=1);

class Weapon
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

**main.php:**

```php
<?php

declare(strict_types=1);

require_once 'Weapon.php';
require_once 'Armor.php';
require_once 'Player.php';
require_once 'Monster.php';

$knight = new Player('Knight', 100, new Weapon('Sword', 25), new Armor('Iron Plate', 10));
$goblin = new Monster('Goblin', 60, 15);
```

`main.php` не знает, как устроен `Weapon` внутри — только подключает файл и использует класс.

В реальных проектах подключением занимается **Composer** — менеджер пакетов для PHP. Захотел библиотеку — одна команда, и сразу пользуешься: никаких `require_once`, классы находятся сами. Мы до него доберёмся позже — пока достаточно `require_once`.

## Массивы объектов

В игре бывает не один монстр, а несколько. В PHP массив объектов — обычный массив:

```php
$horde = [
    new Monster('Goblin', 40, 10),
    new Monster('Orc', 80, 20),
    new Monster('Troll', 120, 15),
];

foreach ($horde as $monster) {
    echo $monster->describe() . "\n";
}
```

## Отряд как объект

Массив персонажей — это отряд. А отряд — тоже объект:

```php
class Party
{
    private array $members = [];

    public function add(Player $player): void
    {
        $this->members[] = $player;
    }

    public function isAlive(): bool
    {
        foreach ($this->members as $member) {
            if ($member->isAlive()) {
                return true;
            }
        }
        return false;
    }

    public function describe(): string
    {
        $lines = [];
        foreach ($this->members as $member) {
            $lines[] = '  ' . $member->describe();
        }
        return 'Отряд:' . "\n" . implode("\n", $lines);
    }
}
```

Снаружи `Party` — один объект: `$party->isAlive()`, `$party->describe()`. Внутри — массив. Это та же композиция из урока 04, но поле теперь коллекция.

## Шпаргалка

- один класс = один `.php` файл
- `require_once 'Weapon.php'` — подключить файл, не более одного раза
- массив объектов: `$horde = [new Monster(...), new Monster(...)]`
- обход: `foreach ($horde as $monster)`
- коллекция — тоже объект: `Party` хранит массив `Player`

## Напоследок

1. В Java один класс = один файл — это правило языка. Файл `Player.java` обязан содержать класс `Player`. В PHP это соглашение, но настолько устоявшееся, что нарушать не принято.

2. Разделение на `.h` и `.cpp` — уникальная особенность C и C++, наследие эпохи, когда компилятору нужно было заранее знать, что существует. В PHP, Python, Java, JavaScript — один файл на класс, без разделения.

3. У каждого языка есть свой менеджер пакетов — инструмент, который скачивает чужой код и подключает его в проект:
   - PHP → **Composer** (`composer require vendor/package`)
   - JavaScript → **npm** (`npm install package`)
   - Python → **pip** (`pip install package`)
   - Java → **Maven** или **Gradle**

   У каждого языка своё: через npm ставятся AI-агенты, фреймворки, библиотеки для работы с API — одной командой, без ручного копирования файлов. Composer для PHP — то же самое.

4. `private array $members` не проверяет, что внутри массива лежат именно `Player`. PHP просто не умеет это выразить без дополнительных инструментов. Поэтому важно добавлять только через `add(Player $player)` — тип гарантирует метод, а не массив.

## Практика

### 6.1. Орда монстров

Раздели классы из прошлых уроков по файлам: `Weapon.php`, `Armor.php`, `Player.php`, `Monster.php`. Подключи их в `main.php` через `require_once`.

Создай массив из 4 монстров. Герой сражается с ними по очереди. После каждого боя — вывод состояния героя. Если HP героя ниже 40 после боя — он пьёт зелье (если есть).

```text
=== Бой 1: Knight vs Goblin ===
Knight: 75/100 HP, Sword (урон: 25), Iron Plate (защита: 10)
=== Бой 2: Knight vs Orc ===
Knight использует зелье.
Knight: 90/100 HP, Sword (урон: 25), Iron Plate (защита: 10)
...
```

### 6.2. Отряд

Напиши класс `Party` в отдельном файле `Party.php`. Отряд хранит произвольное количество игроков.

Методы:
- `add(Player $player): void`
- `isAlive(): bool` — жив ли хоть кто-то
- `describe(): string` — статус всех членов

Проведи бой: отряд из 3 героев против массива из 4 монстров. Каждый ход каждый живой герой атакует текущего монстра. Монстр бьёт первого живого героя.

```php
$party = new Party();
$party->add(new Player('Knight', 100, new Weapon('Sword', 25), new Armor('Iron Plate', 10)));
$party->add(new Player('Mage', 60, new Weapon('Staff', 40), new Armor('Robe', 2)));
$party->add(new Player('Archer', 80, new Weapon('Bow', 25), new Armor('Leather', 5)));

echo $party->describe() . "\n";
```
