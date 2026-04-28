# 04. Объекты владеют объектами

Идею ты уже знаешь из cpp-quest: если «это штука с именем и поведением» — это отдельный объект. Здесь только PHP-синтаксис и задачи.

## C++ → PHP

В C++ объект-поле инициализируется через список инициализации:

```cpp
class Player {
    std::string name;
    int hp;
    Weapon weapon;

public:
    Player(std::string n, int h, Weapon w)
        : name(n), hp(h), weapon(w) {}
};
```

В PHP такого списка нет. Просто передаёшь объект как параметр и сохраняешь:

```php
class Player
{
    public function __construct(
        private string $name,
        private int $hp,
        private Weapon $weapon,
    ) {
    }
}
```

Promoted properties сделают остальное сами. Тип `Weapon` в параметре — PHP проверит, что туда пришёл именно этот объект.

## Рабочий пример

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
        $dmg = $this->weapon->strike();
        $target->takeDamage($dmg);
    }

    public function takeDamage(int $damage): void
    {
        $this->hp -= $damage;

        if ($this->hp < 0) {
            $this->hp = 0;
        }
    }

    public function equip(Weapon $weapon): void
    {
        $this->weapon = $weapon;
    }

    public function isAlive(): bool
    {
        return $this->hp > 0;
    }

    public function describe(): string
    {
        return $this->name . ': ' . $this->hp . ' HP, ' . $this->weapon->describe();
    }
}

$knight = new Player('Knight', 100, new Weapon('Sword', 25));
$rogue = new Player('Rogue', 80, new Weapon('Dagger', 35));

echo $knight->describe() . "\n"; // Knight: 100 HP, Sword (урон: 25)
echo $rogue->describe() . "\n";  // Rogue: 80 HP, Dagger (урон: 35)

$knight->equip(new Weapon('Axe', 40));
echo "\nKnight нашёл Axe!\n";
echo $knight->describe() . "\n";

echo "\n";

$round = 1;
while ($knight->isAlive() && $rogue->isAlive()) {
    echo "--- Ход $round ---\n";
    $knight->attack($rogue);
    if ($rogue->isAlive()) {
        $rogue->attack($knight);
    }
    echo $knight->describe() . "\n";
    echo $rogue->describe() . "\n";
    $round++;
}
```

## Про объекты и копии

В C++ `equip(Weapon w)` без `&` принимал копию специально — чтобы не потерять оружие, если оригинал исчезнет.

В PHP объекты всегда передаются через хэндл (внутренний указатель). Копирование нужно явно: `clone $weapon`. В большинстве задач этот курса это не понадобится.

## Шпаргалка

- `private Weapon $weapon` внутри `Player` — игрок **имеет** оружие
- тип объекта в параметре: `private Weapon $weapon` — PHP проверит при создании
- вызов метода через поле: `$this->weapon->strike()`
- замена объекта-поля: `$this->weapon = $newWeapon;`
- без списка инициализации: promoted properties делают то же самое

## Напоследок

1. В индустрии есть известное правило: «предпочитай композицию наследованию». До наследования мы ещё доберёмся — но заметь, что без него уже можно построить очень много.

2. В игровой индустрии архитектура Entity-Component-System (ECS) — это композиция, доведённая до предела. Персонаж — пустая сущность, к которой прикрепляют компоненты: здоровье, оружие, инвентарь, AI. Каждый компонент — маленький объект.

3. Композиция помогает тестировать. Если `Player` получает `Weapon` снаружи — в тесте можно передать слабое оружие с уроном 0 и проверить, что броня работает. Или передать сильное — и проверить, что персонаж умирает за один удар. Объект, который сам создаёт зависимости внутри себя, так не проверить.

## Практика

### 4.1. Броня

Напиши класс `Armor` с private-свойствами `$name` и `$defense` и методом `defense(): int`.

Добавь `Armor` как поле в `Player`. Метод `takeDamage` должен вычитать защиту брони из входящего урона (минимум 0).

```php
$player = new Player('Knight', 100, new Weapon('Sword', 25), new Armor('Iron Plate', 10));
$player->takeDamage(30);
assert($player->describe() === 'Knight: 80 HP, Sword (урон: 25), Iron Plate (защита: 10)');
```

### 4.2. Зелье

Напиши класс `Potion` с private-свойствами `$name`, `$heal` и `$charges`.

Добавь метод `use(Player $target): void` — лечит цель на `$heal` HP и тратит один заряд. Когда заряды закончились — ничего не делает.

Добавь в `Player` поле `$maxHp`. Задай его равным начальному `$hp` в конструкторе. HP не должно подниматься выше максимума.

```php
$player = new Player('Knight', 100, new Weapon('Sword', 25), new Armor('Shield', 5));
$potion = new Potion('Heal Potion', 30, 2);

$player->takeDamage(50); // 50 - 5 = 45 урона → 55 HP
$potion->use($player);   // +30 → 85 HP
$potion->use($player);   // +30 → 100 HP (не выше максимума)
$potion->use($player);   // заряды кончились, ничего не происходит

assert($player->hp() === 100);
```

Добавь публичный метод `hp(): int` в `Player` для проверки.

