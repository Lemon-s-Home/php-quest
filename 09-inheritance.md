# 09. Объекты наследуют

## Дублирование

Посмотри на `Player` и `Monster` из прошлых уроков:

```php
class Player
{
    private int $maxHp;

    public function __construct(
        private string $name,
        private int $hp,
    ) {
        $this->maxHp = $hp;
    }

    public function takeDamage(int $damage): void
    {
        $this->hp = max(0, $this->hp - $damage);
    }

    public function isAlive(): bool
    {
        return $this->hp > 0;
    }

    public function name(): string
    {
        return $this->name;
    }
}

class Monster
{
    private int $maxHp;

    public function __construct(
        private string $name,
        private int $hp,
    ) {
        $this->maxHp = $hp;
    }

    public function takeDamage(int $damage): void
    {
        $this->hp = max(0, $this->hp - $damage);
    }

    public function isAlive(): bool
    {
        return $this->hp > 0;
    }

    public function name(): string
    {
        return $this->name;
    }
}
```

Одинаковые поля. Одинаковые методы. Нашёл баг в `takeDamage` — чинить в двух местах. Добавишь `Boss` — в трёх.

## Общее — в базовый класс

**Наследование** позволяет вынести общий код в один класс:

```php
class Character
{
    private int $maxHp;

    public function __construct(
        private string $name,
        private int $hp,
    ) {
        $this->maxHp = $hp;
    }

    public function takeDamage(int $damage): void
    {
        $this->hp = max(0, $this->hp - $damage);
    }

    public function isAlive(): bool
    {
        return $this->hp > 0;
    }

    public function name(): string
    {
        return $this->name;
    }

    public function describe(): string
    {
        return $this->name . ': ' . $this->hp . '/' . $this->maxHp . ' HP';
    }
}
```

`Player` и `Monster` теперь **наследуют** от `Character` и добавляют только своё:

```php
class Warrior extends Character
{
    public function __construct(string $name, int $hp, private Weapon $weapon)
    {
        parent::__construct($name, $hp);
    }

    public function attack(Character $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }
}

class Mage extends Character
{
    public function __construct(string $name, int $hp, private int $spellPower)
    {
        parent::__construct($name, $hp);
    }

    public function attack(Character $target): void
    {
        $target->takeDamage($this->spellPower);
    }
}
```

Разберём:
- `extends Character` — наследуем всё от `Character`
- `parent::__construct($name, $hp)` — вызываем конструктор базового класса
- каждый наследник добавляет только своё: `Warrior` — оружие, `Mage` — силу заклинания

## C++ → PHP

В C++ вызов конструктора базового класса — в списке инициализации:

```cpp
Warrior(string name, int hp, Weapon w)
    : Character(name, hp), weapon(w) {}
```

В PHP — явный вызов `parent::__construct()` внутри конструктора:

```php
public function __construct(string $name, int $hp, private Weapon $weapon)
{
    parent::__construct($name, $hp);
}
```

`protected` работает так же: поле или метод видно в самом классе и во всех наследниках, но не снаружи.

## Абстрактный класс

Сейчас `Character` можно создать напрямую:

```php
$c = new Character('Nobody', 100); // работает, но смысла нет
```

Персонаж без конкретного типа — странная сущность. Чтобы запретить это, класс помечают `abstract`:

```php
abstract class Character
{
    // ...
}

$c = new Character('Nobody', 100); // ошибка: нельзя создать абстрактный класс
```

Абстрактный класс может также объявлять методы без реализации — наследник **обязан** их написать:

```php
abstract class Character
{
    // ...

    abstract public function attack(Character $target): void;
}
```

Теперь нельзя создать класс, унаследованный от `Character`, не написав `attack`. PHP выдаст ошибку.

## interface vs abstract class

Оба запрещают создавать объект напрямую и оба могут объявлять методы без реализации. Разница:

| | `interface` | `abstract class` |
|---|---|---|
| Общий код | Нет | Да |
| Свойства | Нет | Да |
| `implements` / `extends` | `implements` | `extends` |
| Несколько сразу | Да (`implements A, B`) | Нет |

Правило простое: если нужно только описать контракт — `interface`. Если нужен общий код или общие свойства — `abstract class`.

## Полный пример

```php
<?php

declare(strict_types=1);

interface Weapon
{
    public function strike(): int;
    public function describe(): string;
}

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

/** Базовый персонаж с именем и HP. Не создаётся напрямую. */
abstract class Character
{
    private int $maxHp;

    public function __construct(
        private string $name,
        private int $hp,
    ) {
        $this->maxHp = $hp;
    }

    abstract public function attack(Character $target): void;

    public function takeDamage(int $damage): void
    {
        $this->hp = max(0, $this->hp - $damage);
    }

    public function isAlive(): bool
    {
        return $this->hp > 0;
    }

    public function name(): string
    {
        return $this->name;
    }

    public function describe(): string
    {
        return $this->name . ': ' . $this->hp . '/' . $this->maxHp . ' HP';
    }
}

/** Воин с оружием. */
class Warrior extends Character
{
    public function __construct(string $name, int $hp, private Weapon $weapon)
    {
        parent::__construct($name, $hp);
    }

    public function attack(Character $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }
}

/** Маг с фиксированной силой заклинания. */
class Mage extends Character
{
    public function __construct(string $name, int $hp, private int $spellPower)
    {
        parent::__construct($name, $hp);
    }

    public function attack(Character $target): void
    {
        $target->takeDamage($this->spellPower);
    }
}

/** Лучник с шансом промаха. */
class Archer extends Character
{
    public function __construct(string $name, int $hp, private int $arrowDamage)
    {
        parent::__construct($name, $hp);
    }

    public function attack(Character $target): void
    {
        if (rand(1, 100) <= 20) {
            echo $this->name() . " промахивается!\n";
            return;
        }
        $target->takeDamage($this->arrowDamage);
    }
}

$party = [
    new Warrior('Knight', 120, new Sword('Sword', 30)),
    new Mage('Merlin', 70, 45),
    new Archer('Robin', 90, 25),
];

$boss = new Warrior('Dragon', 200, new Sword('Claw', 20));

echo "=== Бой с боссом ===\n\n";

while ($boss->isAlive()) {
    foreach ($party as $hero) {
        if ($hero->isAlive()) {
            $hero->attack($boss);
        }
    }

    if ($boss->isAlive()) {
        foreach ($party as $hero) {
            if ($hero->isAlive()) {
                $boss->attack($hero);
            }
        }
    }

    foreach ($party as $hero) {
        echo $hero->describe() . "\n";
    }
    echo $boss->describe() . "\n\n";
}

$survivors = array_filter($party, fn($h) => $h->isAlive());
if (count($survivors) > 0) {
    echo "Победа! Выжили: " . implode(', ', array_map(fn($h) => $h->name(), $survivors)) . "\n";
} else {
    echo "Все пали в бою.\n";
}
```

## Шпаргалка

- `extends` — наследовать от класса
- `parent::__construct(...)` — вызвать конструктор родителя
- `protected` — видно в классе и наследниках, скрыто снаружи
- `abstract class` — нельзя создать напрямую, может содержать общий код
- `abstract public function` — наследник обязан реализовать
- `interface` — только контракт, без кода. `abstract class` — контракт + общий код

```php
abstract class Character
{
    public function __construct(protected string $name, private int $hp) {}
    abstract public function attack(Character $target): void;
    public function isAlive(): bool { return $this->hp > 0; }
}

class Warrior extends Character
{
    public function attack(Character $target): void { /* ... */ }
}
```

## Напоследок

1. В Java все классы неявно наследуют от `Object` — общий предок. В PHP то же самое: любой класс без явного `extends` наследует от встроенного `stdClass`-подобного корня. В C++ такого нет — каждый класс сам по себе.

2. Создатели Go убрали наследование из языка полностью. Только интерфейсы и композиция. Rust поступил так же. Это говорит о том, что индустрия движется от наследования к композиции — но для группировки общего кода наследование по-прежнему удобно.

3. `abstract` в PHP — это то же, что чисто виртуальный метод (`= 0`) в C++. Только в C++ абстрактность определяется наличием хотя бы одного `= 0`, а в PHP — явным словом `abstract` перед классом.

## Практика

### 9.1. Разбойник

Напиши класс `Rogue extends Character`. У разбойника есть шанс уклонения — если «кубик» выпал удачно, `takeDamage` не применяется. Атакует кинжалом с фиксированным уроном.

```php
$rogue = new Rogue('Shadow', 80, 30, 40); // имя, hp, урон, шанс уклонения %

$enemy = new Warrior('Orc', 100, new Sword('Club', 20));
$enemy->attack($rogue); // может не пройти
echo $rogue->describe() . "\n";
```

### 9.2. Отряд vs орда

Создай отряд из `Warrior`, `Mage`, `Rogue`. Создай массив из 3 монстров — любые наследники `Character` с разными характеристиками.

Герои сражаются с монстрами по очереди: сначала весь отряд добивает первого монстра, потом второго, и так далее. Каждый ход монстр бьёт случайного живого героя. После каждого боя выводи состояние отряда.
