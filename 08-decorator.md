# 08. Объекты оборачивают объекты

В прошлом уроке мы научили `Player` принимать любой `Weapon`. Теперь — как добавлять эффекты к оружию, не трогая оригинал.

## Когда хочется добавить эффект

Есть меч. Хочется меч с ядом. Как?

**Вариант 1** — добавить поле в `Sword`:

```php
class Sword implements Weapon
{
    private int $poisonDamage = 0;
    private int $fireDamage = 0;
    private bool $hasCrit = false;
    // ...
}
```

Для каждого нового эффекта — новое поле. Большинство будут нулями. Класс раздувается.

**Вариант 2** — новый класс на каждую комбинацию:

```php
class PoisonedSword { /* копируем весь Sword + добавляем яд */ }
class FirePoisonedSword { /* копируем ещё раз */ }
class CritFirePoisonedSword { /* ... */ }
```

Дублирование, которое растёт как снежный ком.

Оба варианта — тупик.

## Обёртка: добавляем поведение снаружи

Другой подход: не менять `Sword`, а **обернуть** его. Обёртка реализует тот же интерфейс `Weapon` и держит внутри другой `Weapon`:

```php
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
```

```php
$sword = new Sword('Sword', 25);
$poisoned = new Poisoned($sword, 10);

assert($poisoned->strike() === 35);
assert($poisoned->describe() === 'Sword (урон: 25) [яд: +10]');
```

Меч не знает про яд. Яд не знает, как меч считает урон.

## Матрёшка

Потому что обёртка принимает `Weapon`, а не `Sword` — её можно обернуть снова:

```php
$sword = new Sword('Sword', 25);
$poisoned = new Poisoned($sword, 10);
$crit = new CriticalStrike($poisoned, 30);

// при крите: (25 + 10) * 2 = 70
```

Каждый слой видит только `Weapon` — ему не важно, что внутри. Это и есть паттерн **Декоратор**.

## Пример: кузница эффектов

Три варианта оружия из одного меча — разные обёртки, один `Player`:

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

class CriticalStrike implements Weapon
{
    public function __construct(
        private Weapon $base,
        private int $critChance,
    ) {
    }

    public function strike(): int
    {
        $dmg = $this->base->strike();
        if (rand(1, 100) <= $this->critChance) {
            $dmg *= 2;
        }
        return $dmg;
    }

    public function describe(): string
    {
        return $this->base->describe() . ' [крит: ' . $this->critChance . '%]';
    }
}

$sword = new Sword('Sword', 25);

$variants = [
    'Обычный'          => $sword,
    'С ядом'           => new Poisoned($sword, 10),
    'С ядом и критом'  => new CriticalStrike(new Poisoned($sword, 10), 30),
];

foreach ($variants as $label => $weapon) {
    echo $label . ': ' . $weapon->describe() . "\n";
    echo '  Удар: ' . $weapon->strike() . "\n";
}
```

```text
Обычный: Sword (урон: 25)
  Удар: 25
С ядом: Sword (урон: 25) [яд: +10]
  Удар: 35
С ядом и критом: Sword (урон: 25) [яд: +10] [крит: 30%]
  Удар: 35  (или 70 при крите)
```

## Шпаргалка

- Декоратор реализует тот же интерфейс, что и оригинал: `implements Weapon`
- Держит внутри `Weapon`, а не конкретный класс: `private Weapon $base`
- Вызывает оригинал и добавляет своё: `$this->base->strike() + $this->extra`
- Обёртки комбинируются в любом порядке и на любую глубину
- Оригинал не знает про обёртки и не меняется

## Напоследок

1. Паттерн «Декоратор» описан в книге «Design Patterns» (1994) — её написали четверо авторов, которых называют «Банда четырёх» (Gang of Four). Книга до сих пор актуальна.

2. В Java декораторы повсюду в стандартной библиотеке: `new BufferedReader(new FileReader("file.txt"))` — `BufferedReader` оборачивает `FileReader` и добавляет буферизацию. Тот же принцип: один объект внутри другого, тот же интерфейс снаружи.

3. Без интерфейса декоратор не работает — обёртка должна быть неотличима от оригинала для внешнего кода. Именно поэтому мы изучили интерфейсы первыми.

## Практика

### 8.1. Огонь и лёд

Напиши две обёртки: `Fire` и `Frost`, обе `implements Weapon`. Проверь, что комбинируются:

```php
$sword = new Sword('Sword', 20);
$fire = new Fire($sword, 15);
$frostFire = new Frost($fire, 8);

assert($fire->strike() === 35);
assert($frostFire->strike() === 43);
assert($frostFire->describe() === 'Sword (урон: 20) [огонь: +15] [лёд: +8]');
```

### 8.2. Вампиризм

Напиши класс `Axe implements Weapon` и обёртку `Vampiric implements Weapon`. `strike()` возвращает урон как обычно. Дополнительный метод `healAmount(): int` возвращает, сколько HP нужно восстановить владельцу (процент от урона базового оружия).

```php
$axe = new Axe(40); // Axe implements Weapon
$vampiric = new Vampiric($axe, 25); // 25% вампиризм

assert($vampiric->strike() === 40);
assert($vampiric->healAmount() === 10); // 25% от 40
```

Подумай: почему `healAmount()` не входит в интерфейс `Weapon`?
