# 02. ООП в PHP после C++

Классы, объекты, `private`, `public` и методы тебе уже знакомы по C++. Здесь надо только привыкнуть к записи в PHP.

Смотрим короткие примеры, потом сразу практика.

## Класс и свойства

В C++:

```cpp
class Animal {
private:
    std::string name;
    int age;
};
```

В PHP:

```php
class Animal
{
    private string $name;
    private int $age;
}
```

Разница на старте такая:

- у свойства в PHP есть `$`;
- `private` и `public` пишутся перед каждым свойством и методом;
- после класса нет `;`.

## Метод

В C++:

```cpp
void birthday() {
    age++;
}
```

В PHP:

```php
public function birthday(): void
{
    $this->age++;
}
```

Здесь новое:

- метод объявляется через `function`;
- тип возвращаемого значения пишется после `:`;
- внутри класса постоянно встречается `$this`.

## `$this`

В C++ в ваших примерах доступ к полям чаще был неявным:

```cpp
void rename(std::string newName) {
    name = newName;
}
```

В PHP так нельзя. Нужно явно сказать, что это свойство текущего объекта:

```php
public function rename(string $newName): void
{
    $this->name = $newName;
}
```

`$this` — это текущий объект.

Если есть два объекта:

```php
$dog = new Animal('Bobik', 3);
$cat = new Animal('Murzik', 5);
```

то при вызове:

```php
$dog->birthday();
```

внутри метода `$this` — это `$dog`.

А при вызове:

```php
$cat->birthday();
```

внутри того же метода `$this` — уже `$cat`.

## Создание объекта

В C++:

```cpp
Animal dog("Bobik", 3);
```

В PHP:

```php
$dog = new Animal('Bobik', 3);
```

То есть:

- переменная пишется с `$`;
- объект создаётся через `new`.

## Вызов метода

В C++:

```cpp
dog.birthday();
```

В PHP:

```php
$dog->birthday();
```

У объекта доступ к методам и свойствам идёт через `->`.

## Короткий пример целиком

```php
<?php

declare(strict_types=1);

class Animal
{
    private string $name;
    private int $age;

    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }

    public function birthday(): void
    {
        $this->age++;
    }

    public function rename(string $newName): void
    {
        $this->name = $newName;
    }

    public function description(): string
    {
        return $this->name . ', ' . $this->age . ' лет';
    }
}
```

Если создать `new Animal('Bobik', 3)`, вызвать `birthday()`, а потом `rename('Rex')`, то `description()` по шагам даст:

- `Bobik, 3 лет`
- `Bobik, 4 лет`
- `Rex, 4 лет`

## Шпаргалка

- свойство: `private int $age;`
- метод: `public function birthday(): void`
- текущий объект: `$this`
- создание объекта: `$dog = new Animal('Bobik', 3);`
- вызов метода: `$dog->birthday();`

```php
class Animal
{
    private string $name;
    private int $age;

    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }

    public function birthday(): void
    {
        $this->age++;
    }
}
```

## Напоследок

1. В PHP метод почти всегда короче, чем в C++, просто потому что меньше служебного синтаксиса вокруг.
2. В ранних уроках лучше привыкать к методам-действиям: `birthday`, `rename`, `takeDamage`, а не к техническим `setAge`, `setName`, `setHp`.

## Практика

2.1. Напиши класс `Pet` с private-свойствами `name` и `age`, методом `birthday()` и методом `description()`.
   - Создание: `new Pet('Bobik', 3)`
   - После одного `birthday()` строка должна стать `Bobik, 4 лет`

2.2. Напиши класс `Player` с private-свойствами `name` и `hp`, методом `takeDamage(int $damage)` и методом `description()`.
   - Создание: `new Player('Mage', 80)`
   - После `takeDamage(30)` строка должна стать `Mage: 50 HP`
   - После ещё одного `takeDamage(100)` строка должна стать `Mage: 0 HP`

2.3. Напиши класс `Weapon` с private-свойствами `name` и `damage`, методом `strike()` и методом `description()`.
   - Создание: `new Weapon('Sword', 25)`
   - `strike()` должен вернуть `25`
   - `description()` должна вернуть `Sword (урон: 25)`
