# 16. Терминал становится игровым

## Игра говорит с игроком

Пока вывод простой:

```php
echo "Knight атакует Orc\n";
echo "Orc: 45 HP\n";
```

Работает, но быстро становится скучно. В текстовой игре терминал — это весь экран. Через него игрок видит бой, выбирает действия, читает события.

Composer позволяет взять готовую библиотеку для красивого терминального вывода.

## CLImate

Поставим библиотеку:

```bash
composer require league/climate
```

Пример:

```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/vendor/autoload.php';

use League\CLImate\CLImate;

$climate = new CLImate();

$climate->green('Knight готов к бою.');
$climate->red('Orc атакует!');
$climate->yellow('HP мало. Пора пить зелье.');
```

Теперь вывод можно подсвечивать по смыслу.

## Не раскидывай библиотеку по игре

Плохая идея:

```php
$climate->red('Orc атакует!');
```

в каждом классе.

Тогда вся игра будет знать про CLImate. Если потом захочется обычный `echo` или другую библиотеку, придётся менять много файлов.

Лучше спрятать библиотеку за своим объектом.

## GameOutput

```php
<?php

declare(strict_types=1);

namespace Game\Terminal;

use Game\Fighter\Fighter;
use League\CLImate\CLImate;

/** Выводит игровые события в терминал. */
class GameOutput
{
    public function __construct(private CLImate $climate)
    {
    }

    public function startBattle(Fighter $hero, Fighter $enemy): void
    {
        $this->climate->green('Бой начинается!');
        $this->status($hero, $enemy);
    }

    public function status(Fighter $hero, Fighter $enemy): void
    {
        $this->climate->table([
            ['Боец', 'HP'],
            [$hero->name(), $hero->hp()],
            [$enemy->name(), $enemy->hp()],
        ]);
    }

    public function enemyAttack(string $name): void
    {
        $this->climate->red($name . ' атакует!');
    }
}
```

Теперь только `GameOutput` знает про CLImate. Остальная игра говорит: «покажи статус», а не «покрась строку зелёным».

## Ввод игрока

CLImate умеет спрашивать ввод:

```php
$input = $climate->input('Что делать?');
$input->accept(['attack', 'defend', 'potion'], true);

$action = $input->prompt();
```

Игрок увидит:

```text
Что делать? [attack/defend/potion]
```

Если написать что-то другое, CLImate спросит снова.

Можно сделать объект для ввода:

```php
<?php

declare(strict_types=1);

namespace Game\Terminal;

use League\CLImate\CLImate;

/** Спрашивает у игрока следующее действие. */
class PlayerInput
{
    public function __construct(private CLImate $climate)
    {
    }

    public function chooseAction(): string
    {
        $input = $this->climate->input('Что делать?');
        $input->accept(['attack', 'defend', 'potion'], true);

        return $input->prompt();
    }
}
```

## Красивый режим

Ещё у CLImate есть `radio`:

```php
$input = $climate->radio('Что делать?', [
    'attack' => 'Атаковать',
    'defend' => 'Защищаться',
    'potion' => 'Выпить зелье',
]);

$action = $input->prompt();
```

Это выбор стрелками. Но такой режим работает не везде одинаково, поэтому для первой версии игры достаточно `input()->accept(...)`.

## Шпаргалка

- `composer require league/climate` ставит библиотеку для терминала
- `green`, `red`, `yellow` подсвечивают вывод
- `table` показывает данные таблицей
- `input()->accept(...)` принимает только разрешённые ответы
- внешнюю библиотеку лучше прятать за своим объектом
- `GameOutput` выводит события
- `PlayerInput` спрашивает действие

## Напоследок

1. Терминальные игры старше графических. В них вся атмосфера держится на тексте, цветах, паузах и хороших формулировках.

2. Библиотека — это не архитектура. Архитектура в том, что у игры есть свои объекты, а чужой код спрятан внутри маленького слоя.

3. Если однажды игра переедет из терминала в Telegram-бота, `GameOutput` и `PlayerInput` можно будет заменить. Бойцы, оружие и стратегии останутся.

## Практика

### 16.1. Цветной лог боя

Поставь CLImate.

Создай класс `GameOutput`, который умеет:

- зелёным писать начало боя;
- красным писать атаку врага;
- жёлтым писать предупреждение, когда HP героя меньше 30;
- таблицей показывать героя и врага.

В `main.php` проведи один раунд и выведи лог через `GameOutput`.

### 16.2. Выбор действия

Создай класс `PlayerInput`.

Метод:

```php
public function chooseAction(): string
```

должен принимать только три команды:

- `attack`
- `defend`
- `potion`

В `main.php` спроси действие игрока и выведи, что он выбрал.
