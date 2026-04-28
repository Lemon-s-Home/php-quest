# 10. Эффекты на персонаже

## Задание: расширь отряд

Возьми `Character`, `Warrior`, `Mage`, `Archer` из прошлого урока. Добавь три эффекта:

- **Яд** — каждый ход теряет 5 HP после получения урона
- **Берсерк** — атакует на двойной урон, но получает на 50% больше урона
- **Благословение** — лечится на 10 HP каждый раз, когда атакует

Попробуй сделать это через наследование. Создай нужные классы и запусти бой.

---

Сделал? Посмотри, что получилось.

Скорее всего у тебя появились примерно такие классы:

```php
class PoisonedWarrior extends Warrior { ... }
class BerserkWarrior extends Warrior { ... }
class BlessedWarrior extends Warrior { ... }
class PoisonedMage extends Mage { ... }
class BerserkMage extends Mage { ... }
```

А что если воин одновременно отравлен **и** в берсерке? `PoisonedBerserkWarrior`. Отравленный благословлённый маг? `PoisonedBlessedMage`. Три эффекта на трёх классах — уже девять комбинаций. Каждая — отдельный класс с копипастом.

Наследование здесь не масштабируется.

## Пока пауза

Ты только что напоролся на известную проблему. Почитай три примера, подумай — потом смотри решение.

---

**Пример 1. Человек не наследует штаны**

Человек *носит* штаны, *носит* обувь, *носит* куртку. Никто не пишет `class Person extends Pants` — это звучит абсурдно. Но в коде такое встречается постоянно, просто с другими именами.

Если есть выбор — бери композицию. Наследование создаёт жёсткую связь, которую потом тяжело разорвать. Композицию можно поменять в любой момент — просто передать другой объект.

---

**Пример 2. Пингвин не летает**

```php
abstract class Bird
{
    abstract public function fly(): void;
}

class Penguin extends Bird
{
    public function fly(): void
    {
        throw new Exception('Пингвины не летают!');
    }
}
```

Пингвин — птица, это факт. Но код, который получает `Bird` и вызывает `fly()`, рассчитывал что это сработает. Наследование дало обещание — наследник его нарушил.

Это называется **принцип Лисков**: наследник должен вести себя так, чтобы код, написанный под родителя, продолжал работать без сюрпризов. Исключение в `fly()` — сюрприз.

Что если дать птице не метод `fly()`, а объект «способ передвижения»?

```php
class Bird
{
    public function __construct(private Movement $movement) {}
}

$sparrow = new Bird(new Flying());
$penguin = new Bird(new Swimming());
```

Тогда никаких обещаний, которые нельзя выполнить.

---

**Пример 3. Круг наследует эллипс**

Математически круг — частный случай эллипса. Казалось бы, `Circle extends Ellipse` — очевидное решение.

```php
class Ellipse
{
    public function setWidth(int $w): void { ... }
    public function setHeight(int $h): void { ... }
}

class Circle extends Ellipse
{
    // Что делать с setWidth и setHeight?
    // Если разрешить — круг перестаёт быть кругом.
    // Если запретить — нарушаем контракт Ellipse.
}
```

В математике круг является эллипсом. В коде — не обязательно, потому что код про поведение, а не про определения. Это один из самых известных споров в ООП, и однозначного ответа до сих пор нет.

Вопрос не «правильно или нет», а «что ломается и почему».

---

Готов смотреть решение? Оно уже знакомое.

## Знакомый инструмент

В уроке 08 мы добавляли эффекты к оружию через декоратор: обёртка реализует тот же интерфейс и держит внутри оригинал.

Тот же приём работает для персонажа. Нужен интерфейс, который описывает «умеет участвовать в бою»:

```php
interface Fightable
{
    public function attack(Fightable $target): void;
    public function takeDamage(int $damage): void;
    public function isAlive(): bool;
    public function name(): string;
}
```

`Character` реализует `Fightable`. Обёртки тоже реализуют `Fightable` и держат внутри `Fightable`.

## Обёртки-эффекты

```php
/** Эффект яда: при получении урона персонаж теряет дополнительные HP. */
class Poisoned implements Fightable
{
    private int $tickDamage = 5;

    public function __construct(private Fightable $base)
    {
    }

    public function attack(Fightable $target): void
    {
        $this->base->attack($target);
    }

    public function takeDamage(int $damage): void
    {
        $this->base->takeDamage($damage);
        $this->base->takeDamage($this->tickDamage);
    }

    public function isAlive(): bool
    {
        return $this->base->isAlive();
    }

    public function name(): string
    {
        return $this->base->name();
    }

    public function describe(): string
    {
        return $this->base->describe() . ' [отравлен]';
    }
}

/** Эффект берсерка: атакует дважды, получает на 50% больше урона. */
class Berserk implements Fightable
{
    public function __construct(private Fightable $base)
    {
    }

    public function attack(Fightable $target): void
    {
        $this->base->attack($target);
        if ($target->isAlive()) {
            $this->base->attack($target);
        }
    }

    public function takeDamage(int $damage): void
    {
        $this->base->takeDamage((int) ($damage * 1.5));
    }

    public function isAlive(): bool
    {
        return $this->base->isAlive();
    }

    public function name(): string
    {
        return $this->base->name();
    }

    public function describe(): string
    {
        return $this->base->describe() . ' [берсерк]';
    }
}
```

Теперь любой эффект на любом персонаже — одна строка:

```php
$warrior = new Warrior('Knight', 120, new Sword('Sword', 30));

$poisonedBerserkWarrior = new Berserk(new Poisoned($warrior));
```

## Полный пример

```php
<?php

declare(strict_types=1);

/** Оружие, которое умеет наносить удар и описывать себя. */
interface Weapon
{
    public function strike(): int;
    public function describe(): string;
}

/** Конкретный меч с именем и уроном. */
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

/** Участник боя: умеет атаковать, получать урон и сообщать своё имя. */
interface Fightable
{
    public function attack(Fightable $target): void;
    public function takeDamage(int $damage): void;
    public function isAlive(): bool;
    public function name(): string;
}

/** Базовый персонаж с именем и HP. Не создаётся напрямую. */
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

    public function attack(Fightable $target): void
    {
        $target->takeDamage($this->weapon->strike());
    }
}

/** Эффект яда: при получении урона персонаж теряет дополнительные HP. */
class Poisoned implements Fightable
{
    private int $tickDamage = 5;

    public function __construct(private Fightable $base)
    {
    }

    public function attack(Fightable $target): void
    {
        $this->base->attack($target);
    }

    public function takeDamage(int $damage): void
    {
        $this->base->takeDamage($damage);
        $this->base->takeDamage($this->tickDamage);
    }

    public function isAlive(): bool
    {
        return $this->base->isAlive();
    }

    public function name(): string
    {
        return $this->base->name();
    }

    public function describe(): string
    {
        return $this->base->describe() . ' [отравлен]';
    }
}

/** Эффект берсерка: атакует дважды, но получает на 50% больше урона. */
class Berserk implements Fightable
{
    public function __construct(private Fightable $base)
    {
    }

    public function attack(Fightable $target): void
    {
        $this->base->attack($target);
        if ($target->isAlive()) {
            $this->base->attack($target);
        }
    }

    public function takeDamage(int $damage): void
    {
        $this->base->takeDamage((int) ($damage * 1.5));
    }

    public function isAlive(): bool
    {
        return $this->base->isAlive();
    }

    public function name(): string
    {
        return $this->base->name();
    }

    public function describe(): string
    {
        return $this->base->describe() . ' [берсерк]';
    }
}

$warrior = new Warrior('Knight', 120, new Sword('Sword', 30));
$enemy = new Warrior('Orc', 100, new Sword('Club', 20));

// Knight отравлен и в берсерке
$knight = new Berserk(new Poisoned($warrior));

echo $knight->describe() . "\n";
echo $enemy->describe() . "\n\n";

$round = 1;
while ($knight->isAlive() && $enemy->isAlive()) {
    echo "--- Ход $round ---\n";
    $knight->attack($enemy);
    if ($enemy->isAlive()) {
        $enemy->attack($knight);
    }
    echo $knight->describe() . "\n";
    echo $enemy->describe() . "\n\n";
    $round++;
}

echo ($knight->isAlive() ? $knight->name() : $enemy->name()) . " победил!\n";
```

## Декоратор на персонаже vs декоратор на оружии

Принцип тот же — интерфейс, обёртка держит оригинал, добавляет своё. Разница только в том, что оборачивается: оружие или персонаж целиком.

| | Оружие (урок 08) | Персонаж (этот урок) |
|---|---|---|
| Интерфейс | `Weapon` | `Fightable` |
| Базовый класс | `Sword`, `Axe` | `Warrior`, `Mage` |
| Обёртки | `Poisoned`, `Fire` | `Poisoned`, `Berserk` |
| Комбинируются | Да | Да |

## Шпаргалка

- Эффекты на персонаже — тот же декоратор, что и на оружии
- Обёртка реализует `Fightable` и держит внутри `Fightable`
- Любой эффект на любом персонаже: `new Berserk(new Poisoned($warrior))`
- Наследование даёт взрыв классов при комбинации эффектов — декоратор нет

## Напоследок

1. Джо Армстронг, создатель языка Erlang: «Хотел банан — получил гориллу, которая держит банан, и все джунгли в придачу.» Это про наследование ради одного метода: берёшь маленькое, получаешь всё остальное вместе с ним.

2. В играх временные эффекты (баффы, дебаффы) почти всегда реализуются именно так — как обёртки или компоненты, а не как новые классы. Иначе игра с сотней эффектов потребовала бы тысячи классов.

4. Книга «Design Patterns» (1994, «Банда четырёх») открывается принципом: «предпочитай композицию наследованию». Это было написано в эпоху, когда все вокруг строили иерархии наследования по десять уровней и потом не могли их поменять. Паттерны Декоратор и Стратегия из той же книги — по сути побег от этой боли.

## Практика

### 10.1. Благословение

Напиши обёртку `Blessed implements Fightable`. Каждый раз, когда благословлённый персонаж атакует — он восстанавливает 10 HP (но не выше максимума).

Для этого потребуется добавить метод `heal(int $amount): void` в `Character` и интерфейс `Fightable`.

```php
$mage = new Mage('Merlin', 80, 40);
$blessed = new Blessed($mage);

// после атаки mage восстанавливает 10 HP
```

### 10.2. Комбо

Создай трёх персонажей с разными комбинациями эффектов и проведи бой между ними:

```php
$a = new Berserk(new Warrior('Ragnar', 100, new Sword('Axe', 25)));
$b = new Poisoned(new Blessed(new Mage('Merlin', 80, 35)));
$c = new Warrior('Guard', 120, new Sword('Spear', 20));
```

Выведи состояние после каждого хода. Убедись, что все эффекты работают одновременно.
