# 03. Объекты рождаются готовыми

В C++ ты уже видел главную мысль: полуготовый объект — это баг.

В PHP всё так же. Отличается только синтаксис конструктора и пара удобных сокращений.

## Конструктор

В C++:

```cpp
class Player {
private:
    std::string name;
    int hp;

public:
    Player(std::string n, int h) {
        name = n;
        hp = h;
    }
};
```

В PHP:

```php
class Player
{
    private string $name;
    private int $hp;

    public function __construct(string $name, int $hp)
    {
        $this->name = $name;
        $this->hp = $hp;
    }
}
```

Главное отличие: в PHP конструктор всегда называется `__construct`.

## Готовый объект

Нормальная запись такая:

```php
$player = new Player('Mage', 80);
```

То есть объект получает всё нужное сразу при создании.

Не так:

```php
$player = new Player();
// потом отдельно имя
// потом отдельно hp
```

А сразу готовым.

## `$this` внутри конструктора

В конструкторе:

```php
public function __construct(string $name, int $hp)
{
    $this->name = $name;
    $this->hp = $hp;
}
```

здесь важно не перепутать две вещи:

- `$name` и `$hp` справа — это параметры конструктора;
- `$this->name` и `$this->hp` слева — это свойства объекта.

Конструктор получает значения снаружи и записывает их внутрь нового объекта.

## Короткая запись конструктора

В PHP можно написать короче:

```php
class Player
{
    public function __construct(
        private string $name,
        private int $hp,
    ) {
    }
}
```

PHP сам:

- создаст private-свойства;
- положит в них значения из параметров.

То есть это короткая форма для такого же конструктора:

```php
class Player
{
    private string $name;
    private int $hp;

    public function __construct(string $name, int $hp)
    {
        $this->name = $name;
        $this->hp = $hp;
    }
}
```

В PHP это просто удобство: не писать лишний повтор.

## Конструктор и обычные методы

После конструктора класс обычно выглядит так:

```php
class Player
{
    public function __construct(
        private string $name,
        private int $hp,
    ) {
    }

    public function takeDamage(int $damage): void
    {
        $this->hp -= $damage;

        if ($this->hp < 0) {
            $this->hp = 0;
        }
    }

    public function description(): string
    {
        return $this->name . ': ' . $this->hp . ' HP';
    }
}
```

Если создать `new Player('Mage', 80)` и вызвать `takeDamage(30)`, `description()` должна вернуть `Mage: 50 HP`.

## Деструктор

В PHP, как и в C++, есть деструктор:

```php
class Player
{
    public function __destruct()
    {
        echo "destroyed\n";
    }
}
```

Он называется `__destruct()`.

Но в PHP это пока не центральная тема, потому что памятью вручную управлять не нужно.

## Сборка мусора

Для начала достаточно помнить простую модель:

- объект живёт, пока на него есть ссылки;
- если ссылок не осталось, PHP может его удалить;
- перед удалением может вызваться `__destruct()`.

Короткий пример:

```php
$weapon = new Weapon('Sword', 25);
unset($weapon);
```

`unset($weapon)` убирает ссылку из переменной. Если других ссылок на объект нет, PHP может его очистить.

## Шпаргалка

- конструктор: `public function __construct(...)`
- запись в свойство: `$this->name = $name;`
- короткий конструктор: `private string $name` прямо в параметрах
- деструктор: `public function __destruct()`
- объект в PHP очищается самим runtime

```php
class Player
{
    public function __construct(
        private string $name,
        private int $hp,
    ) {
    }
}
```

## Напоследок

1. Короткий конструктор в PHP похож на очень удобную шпаргалку: смысл тот же, просто меньше шума.
2. Деструктор в PHP знать полезно, но в обычном коде ты о нём думаешь намного реже, чем в C++.

## Практика

3.1. Напиши класс `Pet` с короткой записью конструктора, private-свойствами `name` и `energy` и методом `description()`.
   - Создание: `new Pet('Bobik', 40)`
   - `description()` должна вернуть `Bobik: 40 energy`

3.2. Напиши класс `Weapon` с длинной записью конструктора: свойства отдельно, потом `__construct`, потом присваивания через `$this`.
   - Создание: `new Weapon('Sword', 25)`
   - Метод `strike()` должен вернуть `25`
   - `description()` должна вернуть `Sword (урон: 25)`

3.3. Напиши класс `Player` с короткой записью конструктора, private-свойствами `name` и `hp`, методом `takeDamage(int $damage)` и методом `description()`.
   - Создание: `new Player('Mage', 80)`
   - После `takeDamage(30)` строка должна стать `Mage: 50 HP`
   - После ещё одного `takeDamage(100)` строка должна стать `Mage: 0 HP`
