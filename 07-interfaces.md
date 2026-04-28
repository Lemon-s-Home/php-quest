# 07. Объекты дают обещания

## Проблема: все умеют бить, но тип разный

У нас несколько классов оружия:

```php
Sword          → strike(): int
Poisoned → strike(): int
CriticalStrike → strike(): int
```

Все умеют `strike()`. Но `Player` хранит `Sword` — и принять другой класс не может:

```php
class Player
{
    public function __construct(
        private string $name,
        private int $hp,
        private Sword $weapon, // только Sword, не Poisoned
    ) {
    }
}
```

Хочется сказать: «мне всё равно, что это за объект — лишь бы умел `strike()`».

## Интерфейс: обещание уметь

В PHP для этого есть `interface`:

```php
interface Weapon
{
    public function strike(): int;
    public function describe(): string;
}
```

Интерфейс описывает **что объект умеет**, не говоря **как**. Тела методов нет — только сигнатуры.

Создать объект интерфейса нельзя:

```php
$w = new Weapon(); // ошибка
```

Интерфейс — это обещание. Чтобы его выполнить, класс пишет `implements`:

```php
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

Если класс объявил `implements Weapon`, но не написал один из методов — PHP выдаст ошибку. Обещание обязательно к исполнению.

## C++ → PHP

В C++ интерфейс — это абстрактный класс с чисто виртуальными методами:

```cpp
class Weapon {
public:
    virtual int strike() const = 0;
    virtual ~Weapon() {}
};

class Sword : public Weapon {
public:
    int strike() const override { return damage; }
};
```

В PHP — отдельное ключевое слово, синтаксис чище:

```php
interface Weapon
{
    public function strike(): int;
}

class Sword implements Weapon
{
    public function strike(): int { return $this->damage; }
}
```

`virtual`, `= 0`, `override` — всего этого в PHP нет. PHP сам знает, что `implements` обязывает реализовать все методы.

## Player принимает любое оружие

Теперь `Player` хранит не `Sword`, а `Weapon`:

```php
class Player
{
    public function __construct(
        private string $name,
        private int $hp,
        private Weapon $weapon,
    ) {
    }

    public function attack(Player $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }
}
```

Игроку всё равно, что у него в руках. Он знает одно: оружие умеет `strike()`.

## Это и есть полиморфизм

Один вызов `$this->weapon->strike()` — но за ним может стоять обычный меч, отравленный, или любой другой класс, реализующий `Weapon`. Код `Player::attack` написан один раз и работает с любым оружием, которое ещё не создано.

## Рабочий пример

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

class Poisoned implements Weapon
{
    public function __construct(
        private Weapon $base,
        private int $poisonDamage,
    ) {
    }

    public function strike(): int
    {
        return $this->base->strike() + $this->poisonDamage;
    }

    public function describe(): string
    {
        return $this->base->describe() . ' [яд: +' . $this->poisonDamage . ']';
    }
}

class Player
{
    private int $maxHp;

    public function __construct(
        private string $name,
        private int $hp,
        private Weapon $weapon,
    ) {
        $this->maxHp = $hp;
    }

    public function attack(Player $target): void
    {
        $target->takeDamage($this->weapon->strike());
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
        return $this->name . ': ' . $this->hp . '/' . $this->maxHp . ' HP, '
            . $this->weapon->describe();
    }
}

$sword = new Sword('Sword', 25);
$poisoned = new Poisoned($sword, 10);

$knight = new Player('Knight', 100, $poisoned);
$rogue = new Player('Rogue', 80, new Sword('Dagger', 35));

$round = 1;
while ($knight->isAlive() && $rogue->isAlive()) {
    echo "--- Ход $round ---\n";
    $knight->attack($rogue);
    if ($rogue->isAlive()) {
        $rogue->attack($knight);
    }
    echo $knight->describe() . "\n";
    echo $rogue->describe() . "\n\n";
    $round++;
}

echo ($knight->isAlive() ? $knight->name() : $rogue->name()) . " победил!\n";
```

## Шпаргалка

- `interface` — обещание: что объект умеет, без реализации
- `implements` — класс выполняет обещание, обязан реализовать все методы
- тип в параметре: `Weapon $weapon` — принимает любой класс с `implements Weapon`
- нельзя: `new Weapon()` — интерфейс не создаётся
- полиморфизм: один вызов `strike()`, разное поведение в зависимости от объекта

```php
interface Weapon
{
    public function strike(): int;
}

class Sword implements Weapon
{
    public function strike(): int { return $this->damage; }
}

class Poisoned implements Weapon
{
    public function __construct(private Weapon $base, private int $extra) {}
    public function strike(): int { return $this->base->strike() + $this->extra; }
}
```

## Напоследок

1. В Java `interface` — отдельное ключевое слово, как в PHP. В C++ интерфейса нет — его имитируют абстрактным классом с `= 0`. PHP здесь ближе к Java, чем к C++.

2. В Go и TypeScript интерфейсы «утиные»: если объект имеет метод `strike()` — он автоматически считается реализующим `Weapon`, без явного `implements`. В PHP и Java привязка явная — пишешь `implements` или ничего не работает.

3. Слово «полиморфизм» с греческого — «много форм». Один и тот же вызов `strike()`, но за ним может стоять меч, отравленный меч или любой будущий класс. Термин придумали биологи — для организмов, которые меняют форму.

## Практика

### 7.1. Ледяное оружие

Напиши класс `Frost implements Weapon`. При ударе добавляет фиксированный урон льдом.

```php
$sword = new Sword('Ice Sword', 20);
$frost = new Frost($sword, 8);

assert($frost->strike() === 28);
assert($frost->describe() === 'Ice Sword (урон: 20) [лёд: +8]');
```

### 7.2. Интерфейс Fightable

Создай интерфейс `Fightable`:

```php
interface Fightable
{
    public function takeDamage(int $damage): void;
    public function isAlive(): bool;
    public function name(): string;
}
```

Пусть `Player` и `Monster` реализуют `Fightable`. Измени `attack` в `Player` — пусть принимает `Fightable`, а не `Player`. Теперь игрок может бить и монстра, и другого игрока одним методом.

```php
$hero = new Player('Knight', 100, new Sword('Sword', 25));
$goblin = new Monster('Goblin', 40, 10);

$hero->attack($goblin);
assert(!$goblin->isAlive() || $goblin->name() === 'Goblin');
```
