# 01. Возвращаемся в PHP

После C++ ты уже серьёзный пёс: бывал в таких подворотнях, что PHP теперь выглядит подозрительно цивилизованно.

Здесь будет тише: не нужно разносить класс по нескольким файлам, следить, что где объявлено, и продираться через тяжёлый синтаксис. У PHP тоже есть свои скелеты в шкафу, но они хотя бы не вылезают каждые пять минут.

Но идея та же: программа хранит данные, вызывает функции, меняет состояние и печатает результат. Перед переходом к классам быстро вспомним основу и спокойно займёмся самими идеями ООП.

## Как запускать файл

Создай новый файл урока, например `01-echo.php`:

```php
<?php

declare(strict_types=1);

echo "Hello, PHP!\n";
```

Запуск:

```bash
php 01-echo.php
```

`<?php` открывает PHP-код.

`declare(strict_types=1);` говорит: хотим более строгую проверку типов.

`echo` печатает текст на экран.

## Переменные

В PHP переменные начинаются с `$`:

```php
<?php

declare(strict_types=1);

$name = 'Archer';
$hp = 120;
$mana = 35;
$isAlive = true;
```

Тип переменной PHP определяет сам по значению.

В C++ было так:

```cpp
std::string name = "Archer";
int hp = 120;
bool isAlive = true;
```

В PHP тип у самой переменной не пишется, но это не значит, что про типы можно не думать.

## Строки

Строки в PHP удобные: их можно склеивать, измерять и искать по ним.

```php
$name = 'Archer';
$title = 'the Swift';

$fullName = $name . ' ' . $title;

echo $fullName . "\n";
echo strlen($name) . "\n";
```

### Одинарные и двойные кавычки

```php
$name = 'Archer';

echo 'Name: $name' . "\n";
echo "Name: $name\n";
```

Результат:

```text
Name: $name
Name: Archer
```

В двойных кавычках PHP подставляет переменные. В одинарных нет.

### Полезные функции для строк

```php
$phrase = '  dragon sword  ';

echo trim($phrase) . "\n";
echo strtoupper($phrase) . "\n";
echo str_contains($phrase, 'sword') ? "yes\n" : "no\n";
```

- `trim()` убирает пробелы по краям
- `strtoupper()` переводит строку в верхний регистр
- `str_contains()` проверяет, есть ли подстрока

## Массивы

В PHP один тип массива покрывает и список, и словарь.

### Обычный массив

```php
$items = ['меч', 'щит', 'зелье'];

echo $items[0] . "\n";
echo count($items) . "\n";
```

Добавить элемент:

```php
$items[] = 'лук';
```

Перебор:

```php
foreach ($items as $item) {
    echo $item . "\n";
}
```

### Ассоциативный массив

```php
$player = [
    'name' => 'Archer',
    'hp' => 120,
    'mana' => 35,
];

echo $player['name'] . "\n";
echo $player['hp'] . "\n";
```

Это похоже на объект, но пока это просто структура данных. В следующих уроках как раз разберём, почему массив и объект — не одно и то же.

## Условия и циклы

Здесь всё почти как раньше:

```php
$hp = 40;

if ($hp > 0) {
    echo "Персонаж жив\n";
} else {
    echo "Персонаж побеждён\n";
}
```

```php
for ($i = 1; $i <= 3; $i++) {
    echo "Удар $i\n";
}
```

```php
foreach (['меч', 'щит', 'лук'] as $item) {
    echo $item . "\n";
}
```

## Функции

Функции объявляются через `function`.

```php
function dealDamage(int $attack, int $armor): int
{
    $damage = $attack - $armor;

    if ($damage < 0) {
        return 0;
    }

    return $damage;
}

echo dealDamage(40, 25) . "\n";
```

Здесь:

- `int $attack` и `int $armor` — типы аргументов
- `: int` — тип возвращаемого значения
- `return` отдаёт результат наружу

Можно возвращать и строку:

```php
function buildTitle(string $name, string $class): string
{
    return $name . ' the ' . $class;
}
```

## Полезный шаблон файла

Почти все примеры пока можно писать так:

```php
<?php

declare(strict_types=1);

function main(): void
{
    $name = 'Archer';
    echo $name . "\n";
}

main();
```

Отдельная `main()` полезна просто потому, что код становится аккуратнее.

## Шпаргалка

| Задача | PHP |
|---|---|
| Объявить переменную | `$hp = 100;` |
| Объявить строку | `$name = 'Archer';` |
| Склеить строки | `$full = $name . ' the Brave';` |
| Длина строки | `strlen($name)` |
| Проверить подстроку | `str_contains($text, 'boss')` |
| Создать массив | `$items = ['меч', 'щит'];` |
| Добавить в массив | `$items[] = 'лук';` |
| Размер массива | `count($items)` |
| Перебрать массив | `foreach ($items as $item)` |
| Объявить функцию | `function heal(int $hp): int` |

## Практика

В этот раз задания лучше делать как маленькие функции и сразу проверять через `assert()`.

Это хороший переход к будущим тестам:

- функция получает данные
- функция возвращает результат
- `assert()` проверяет, что результат правильный

Если `assert()` молчит, значит проверка прошла.

### 1.1. Полное имя героя

Напиши функцию:

```php
function buildHeroTitle(string $name, string $class): string
```

Она должна возвращать строку вида:

```text
Артём the Лучник
```

Пример заготовки:

```php
<?php

declare(strict_types=1);

function buildHeroTitle(string $name, string $class): string
{
    // TODO
}

assert(buildHeroTitle('Артём', 'Лучник') === 'Артём the Лучник');
assert(buildHeroTitle('Лена', 'Маг') === 'Лена the Маг');
assert(buildHeroTitle('Игорь', 'Паладин') === 'Игорь the Паладин');
```

Что здесь повторяем:

- аргументы функции
- возврат строки
- склейку через `.`

### 1.2. Рюкзак героя

Напиши функцию:

```php
/**
 * @param array<int, string> $items
 */
function backpackSummary(array $items): string
```

Она должна возвращать строку вида:

```text
В рюкзаке: меч, щит, зелье
```

Если массив пустой, вернуть:

```text
В рюкзаке: пусто
```

Подсказка: пригодится `implode(', ', $items)`.

Пример заготовки:

```php
<?php

declare(strict_types=1);

/**
 * @param array<int, string> $items
 */
function backpackSummary(array $items): string
{
    // TODO
}

assert(backpackSummary(['меч', 'щит', 'зелье']) === 'В рюкзаке: меч, щит, зелье');
assert(backpackSummary(['лук']) === 'В рюкзаке: лук');
assert(backpackSummary([]) === 'В рюкзаке: пусто');
```

Что здесь повторяем:

- массивы
- `count()`
- `implode()`
- `if`

### 1.3. Функция урона

Напиши функцию:

```php
function dealDamage(int $attack, int $armor): int
```

Правила:

- урон = атака минус броня
- если получилось меньше нуля, вернуть `0`

Пример заготовки:

```php
<?php

declare(strict_types=1);

function dealDamage(int $attack, int $armor): int
{
    // TODO
}

assert(dealDamage(40, 25) === 15);
assert(dealDamage(10, 30) === 0);
assert(dealDamage(55, 5) === 50);
```

Что здесь повторяем:

- числа
- условия
- `return`

## Как запускать проверки

Просто запускай файл как обычно:

```bash
php 01-functions.php
```

Если какой-то `assert()` не пройдёт, PHP остановится на ошибке.

## Напоследок

1. Многие большие системы много лет работают на PHP: Facebook, Slack, Wikipedia, PorhHub. Это неплохо ломает миф, что PHP годится только для маленьких сайтов.
2. PHP считают удобным языком для изучения ООП: порог входа низкий, а объектная модель — серьёзная. В популярных Go, Python с ООП всё заметно проще и слабее.
