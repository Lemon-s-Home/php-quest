# 05. Объекты сами за себя отвечают

Принцип ты уже знаешь: «Скажи, не спрашивай». Объект сам следит за своими данными. Здесь — только то, что в PHP выглядит иначе, чем в C++.

## Геттеры без `get`

В C++ поле и метод — оба члены класса, имена не могут совпадать:

```cpp
class Player {
    int hp;
    int hp() const; // ошибка — имя уже занято
};
```

Поэтому в C++ геттеры писали `getHp()`, `getName()`.

В PHP поля и методы живут в разных пространствах внутри объекта — конфликта нет:

```php
class Player
{
    private int $hp;

    public function hp(): int // ок
    {
        return $this->hp;
    }
}
```

Поэтому в PHP принято называть геттер как само поле — без `get`. Это не просто соглашение, это стало возможным именно потому, что язык позволяет.

Правило осталось прежним: **геттер для чтения — нормально. Сеттер — подумай, не нужен ли метод с поведением.** Если внешний код делает `$player->setHp($player->hp() - 10)` — нужен `takeDamage(10)`.

## `__toString()`

В C++ для вывода через `cout` перегружали `operator<<` — это отдельная функция с `friend`, потому что левая часть `cout` не наш класс.

В PHP такой проблемы нет. Есть магический метод `__toString()` — он вызывается автоматически, когда объект используется как строка:

```php
class Weapon
{
    public function __construct(
        private string $name,
        private int $damage,
    ) {
    }

    public function __toString(): string
    {
        return $this->name . ' (урон: ' . $this->damage . ')';
    }
}

$sword = new Weapon('Sword', 25);
echo $sword;             // Sword (урон: 25)
echo "Оружие: $sword\n"; // тоже работает
```

Удобно, но у неявного вызова есть цена: читая `echo $sword`, не видно сразу, что именно выведется. Явный `echo $sword->describe()` понятнее.

В этом курсе используем `describe()`. `__toString()` стоит знать, но не злоупотреблять.

## Шпаргалка

- геттер в PHP: `public function hp(): int` — без `get` в имени
- сеттер — почти всегда сигнал, что нужен метод с поведением
- `__toString()` — вызывается при `echo $object` и в строках `"$object"`
- явный `describe()` понятнее неявного `__toString()`

## Напоследок

1. `__toString()` в PHP — то же, что `__str__` в Python и `toString()` в Java. Везде идея одна: объект сам решает, как себя показать. Везде явный метод понятнее магии.

2. Хороший признак инкапсуляции: если убрать все геттеры из класса, большинство кода продолжает работать. Если сломалось много — внешний код слишком много знал о внутреннем устройстве объекта.

## Практика

### 5.1. Монстр

Напиши класс `Monster` с private-свойствами `$name`, `$hp` и `$damage`.

Методы:
- `attack(Player $target): void`
- `takeDamage(int $damage): void`
- `isAlive(): bool`
- `describe(): string` — `Goblin: 60/60 HP, урон: 15`

```php
$hero = new Player('Knight', 100, new Weapon('Sword', 25), new Armor('Iron Plate', 10));
$goblin = new Monster('Goblin', 60, 15);

while ($hero->isAlive() && $goblin->isAlive()) {
    $hero->attack($goblin);
    if ($goblin->isAlive()) {
        $goblin->attack($hero);
    }
    echo $hero->describe() . "\n";
    echo $goblin->describe() . "\n---\n";
}
```

