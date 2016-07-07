Прочитав [статью](http://alexvaleev.ru/orm-d7/) Алексея Валеева понимаешь насколько не хватает типовых примеров работы с ORM в BitrixFramework. Пополним коллекцию примеров с неочевидной логикой.

* [Пример 1](#Пример-1)
* [Пример 2](#Пример-2)

#Пример 1

Джоин Bitrix\Iblock\ElementTable по нескольким условиям

```php
$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());
$query
    ->registerRuntimeField('element', [
            'data_type' => 'Bitrix\Iblock\ElementTable',
            'reference' => [
                '=this.ID' => 'ref.IBLOCK_ID',
                //добавим условие что нам нужны элементы с определеннм названием
                '=ref.NAME' => new Bitrix\Main\DB\SqlExpression('?', 'Some element name'),
            ],
        ]
    )
    ->setSelect([
        'ID',
        'element',
    ]);

```

#Пример 2

Выберем инфоблоки с названием News у которых есть хотя бы один элемент. При этом в выборке каждая строка должна соответствовать уникальному инфоблоку, т.е. инфоблоки в выборке не должны дублироваться.


```php
$query = new \Bitrix\Main\Entity\Query(Bitrix\Iblock\IblockTable::getEntity());
$query
    ->registerRuntimeField('element', [
            'data_type' => 'Bitrix\Iblock\ElementTable',
            'reference' => [
                '=this.ID' => 'ref.IBLOCK_ID',
            ],
        ]
    )
    //регистрируем поле с минимальным ID эл-та (нам не важно какой именно этот элемент, важно есть ли в принципе минимальный ID или нет)
    ->registerRuntimeField('min_element_id', [
        'data_type'=>'integer',
        'expression' => ['MIN(%s)', 'element.ID']
    ])
    //фильтруем
    ->setFilter([
        '!min_element_id' => false, 
        'NAME' => 'News'
    ])
    //группируем по ID инфоблока
    ->setGroup([
        'ID',
    ]);

```

Наступил на грабли в строке 
```php
'expression' => ['MIN(%s)', 'element.ID']
```
тут нужно быть внимательным.

Например, если указать
```php
'expression' => ['MIN(element.ID)']
```
эта конструкция работать не будет т.к. в условие HAVING sql-запроса попадет именно строка ```MIN(element.ID)```. 

И по скольку битрикс использует свои алиасы для полей и таблиц - таблица element не будет найдена. Соответственно для того чтобы битрикс корректно заменил element.ID на нужный алиас - это поле необходимо передавать вторым элементом в массиве expression.
