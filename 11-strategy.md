# 11. Объекты меняют поведение

## Декоратор vs стратегия

Декоратор *добавляет* поведение поверх существующего — яд, берсерк, благословение. Оригинал остаётся, обёртка дополняет.

Стратегия *заменяет* поведение целиком. Персонаж перестаёт бить мечом и начинает стрелять. Не дополняет — переключается.

## Проблема: поведение зашито намертво

Воин атакует мечом. Но что если он поднял лук? Или перешёл в оборону?

```php
public function attack(Fightable $target): void
{
    if ($this->style === 'melee') {
        $target->takeDamage($this->weapon->strike());
    } elseif ($this->style === 'ranged') {
        $target->takeDamage($this->arrowDamage);
    } elseif ($this->style === 'defensive') {
        $target->takeDamage(5);
        $this->heal(10);
    }
    // каждый новый стиль — новый elseif
}
```

Метод `attack` знает про все стили и растёт с каждым новым. Добавить стиль — значит залезть внутрь и поменять код.

## Стиль атаки — отдельный объект

Выносим поведение в интерфейс:

```php
interface AttackStyle
{
    public function execute(Fightable $attacker, Fightable $target): void;
}
```

Конкретные стили:

```php
class MeleeAttack implements AttackStyle
{
    public function __construct(private Weapon $weapon)
    {
    }

    public function execute(Fightable $attacker, Fightable $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }
}

class RangedAttack implements AttackStyle
{
    private int $arrows;

    public function __construct(private int $damage, int $arrows)
    {
        $this->arrows = $arrows;
    }

    public function execute(Fightable $attacker, Fightable $target): void
    {
        if ($this->arrows <= 0) {
            echo $attacker->name() . ": стрелы кончились!\n";
            return;
        }
        $this->arrows--;
        $target->takeDamage($this->damage);
    }
}
```

Персонаж хранит текущий стиль и делегирует ему атаку:

```php
class Warrior extends Character
{
    private AttackStyle $style;

    public function __construct(string $name, int $hp, AttackStyle $style)
    {
        parent::__construct($name, $hp);
        $this->style = $style;
    }

    public function setStyle(AttackStyle $style): void
    {
        $this->style = $style;
    }

    public function attack(Fightable $target): void
    {
        $this->style->execute($this, $target);
    }
}
```

Один вызов в `attack`. Стиль решает всё. Смена стиля — одна строка:

```php
$warrior->setStyle(new RangedAttack(20, 10));
```

## Полный пример

```php
<?php

declare(strict_types=1);

/** Оружие, которое наносит удар. */
interface Weapon
{
    public function strike(): int;
}

/** Конкретный меч. */
class Sword implements Weapon
{
    public function __construct(private int $damage)
    {
    }

    public function strike(): int
    {
        return $this->damage;
    }
}

/** Участник боя. */
interface Fightable
{
    public function attack(Fightable $target): void;
    public function takeDamage(int $damage): void;
    public function heal(int $amount): void;
    public function isAlive(): bool;
    public function name(): string;
    public function describe(): string;
}

/** Стиль атаки — поведение, которое можно подменить. */
interface AttackStyle
{
    public function execute(Fightable $attacker, Fightable $target): void;
}

/** Удар в ближнем бою. */
class MeleeAttack implements AttackStyle
{
    public function __construct(private Weapon $weapon)
    {
    }

    public function execute(Fightable $attacker, Fightable $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }
}

/** Стрельба из лука с ограниченным запасом стрел. */
class RangedAttack implements AttackStyle
{
    private int $arrows;

    public function __construct(private int $damage, int $arrows)
    {
        $this->arrows = $arrows;
    }

    public function execute(Fightable $attacker, Fightable $target): void
    {
        if ($this->arrows <= 0) {
            echo $attacker->name() . ": стрелы кончились!\n";
            return;
        }
        $this->arrows--;
        $target->takeDamage($this->damage);
    }
}

/** Оборонительная стойка: слабый удар и небольшое лечение. */
class DefensiveStance implements AttackStyle
{
    public function execute(Fightable $attacker, Fightable $target): void
    {
        $target->takeDamage(8);
        $attacker->heal(12);
    }
}

/** Базовый персонаж. */
abstract class Character implements Fightable
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

    public function heal(int $amount): void
    {
        $this->hp = min($this->maxHp, $this->hp + $amount);
    }

    public function isAlive(): bool
    {
        return $this->hp > 0;
    }

    public function name(): string
    {
        return $this->name;
    }

    public function hp(): int
    {
        return $this->hp;
    }

    public function describe(): string
    {
        return $this->name . ': ' . $this->hp . '/' . $this->maxHp . ' HP';
    }
}

/** Воин с переключаемым стилем атаки. */
class Warrior extends Character
{
    private AttackStyle $style;

    public function __construct(string $name, int $hp, AttackStyle $style)
    {
        parent::__construct($name, $hp);
        $this->style = $style;
    }

    public function setStyle(AttackStyle $style): void
    {
        $this->style = $style;
    }

    public function attack(Fightable $target): void
    {
        $this->style->execute($this, $target);
    }
}

$sword = new Sword(30);
$melee = new MeleeAttack($sword);
$ranged = new RangedAttack(20, 3);
$defensive = new DefensiveStance();

$knight = new Warrior('Knight', 100, $ranged);
$troll = new Warrior('Troll', 180, $melee);

echo $knight->describe() . "\n";
echo $troll->describe() . "\n\n";

$round = 1;
while ($knight->isAlive() && $troll->isAlive()) {
    echo "--- Ход $round ---\n";

    // тактика: стрелы — потом меч — при низком HP оборона
    if ($knight->hp() < 40) {
        $knight->setStyle($defensive);
    } else {
        $knight->setStyle($ranged); // ranged сам скажет если стрелы кончились
    }

    $knight->attack($troll);
    if ($troll->isAlive()) {
        $troll->attack($knight);
    }

    echo $knight->describe() . "\n";
    echo $troll->describe() . "\n\n";
    $round++;
}

echo ($knight->isAlive() ? $knight->name() : $troll->name()) . " победил!\n";
```

## Шпаргалка

- Стратегия — поведение как отдельный объект, который можно подменить в рантайме
- `setStyle(AttackStyle $style)` — смена поведения одной строкой
- `attack()` не знает деталей — только вызывает `$this->style->execute(...)`
- Декоратор дополняет. Стратегия заменяет целиком.
- Каждый стиль — маленький класс с одной ответственностью

## Напоследок

1. Паттерн «Стратегия» — один из самых используемых в индустрии. В игровом AI стратегия определяет поведение монстра: агрессивный, осторожный, убегающий. Монстр переключается между ними по ситуации — без единого `if` в основном коде.

2. Есть неформальное правило: «три `if` по типу — скорее всего скрытая стратегия».

```php
if ($type === 'card') ...
if ($type === 'paypal') ...
if ($type === 'crypto') ...
```

Каждый новый способ оплаты — новый `if`. Стратегия убирает их все: просто передаёшь нужный объект.

Наследование говорит: «объект уже родился таким». Стратегия говорит: «объект может менять поведение».

3. Шахматные движки (Stockfish, Leela) построены на том же принципе: алгоритм оценки позиции — подменяемый объект. Это позволяет сравнивать разные стратегии оценки без изменения основного кода.

## Практика

### 11.1. Стиль монстра

Создай монстра, который меняет стратегию в зависимости от HP. Выше 50% — `AggressiveAttack` (высокий урон). Ниже 50% — `DesperateAttack` (очень высокий урон, но монстр ранит себя).

Монстр сам переключает стиль в начале каждого хода.

### 11.2. Два бойца, разные тактики

Создай двух воинов с разными начальными стилями. Каждый меняет тактику по ситуации — сам решает когда и на что. Проведи бой и выведи какой стиль использовался каждый ход.
