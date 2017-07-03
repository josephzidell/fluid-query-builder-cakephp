# fluid-query-builder-cakephp
Ever wanted to write queries in CakePHP fluidly?
```php
// nice
Model->id(1)->get()

// ugly
Model->find('first', ['conditions' => ['Model.id' => 1]])
```


## Requirements
* PHP 5.6+
* CakePHP 1.x or 2.x
* tightenco/collect


## Installation

With [Composer](https://getcomposer.org):
```bash
composer install josephzidell/fluid-query-builder-cakephp
```


## Documentation
### Three parts:
1. Building
2. Fetching
3. Tuning

#### 1. Building: common scenarios
|Method|Does|Notes|
|---|---|---|
|`id(1)`||Shortcut to `where()`|
|<ul><li>`fields('sku')`</li><li>`fields('sku', 'price', 'name')`</li><li>`fields(['sku', 'price'])`</li></ul>|certain fields|<ul><li>Array or [variadic](http://php.net/manual/en/functions.arguments.php#functions.variable-arg-list)</li><li>Is stackable</li></ul>|
|<ul><li>`where(['sku' => '001-0003'])`</li><li>`where('sku = "001-0003"')`</li></ul>|conditions|<ul><li>Array or string</li><li>is stackable</li></ul>|
|`contain(['OtherModel'])`|contain||
|<ul><li>`orderBy('id') // id ASC`</li><li>`orderBy('id DESC')`</li><li>`orderBy(['id' => 'DESC'])`</li><li>`orderBy(['id' => 'DESC', 'price' => 'ASC']) // id DESC, price ASC`</li><li>`orderBy(['id' => 'DESC'])->orderBy('price') // same as previous`</li></ul>|order|<ul><li>Array or string</li><li>is stackable</li></ul>|
|`limit(2)->offset(4)`|limit && offset|may be used separately|
|`page(2, 50)`|page|page 2, page size 50|
|<ul><li>`random()->getFirst() // gets one`</li><li>`random()->limit(5)->get() // gets 5`</li></ul>|random|Shortcut for `orderBy('rand()')`|
|`neighbors(1, 'id')`|neighbors|Pro tip: combine with `where()` to step through search results|

#### 2. Fetching
```php
// list
fields('id', 'sku')->get() // [1 => 'First record', 2 => 'Second record']

// count
where(['sku LIKE' => '001-%'])->count() // 5

// chunking
where(['to_export' => true])->chunk(100, function (\Illuminate\Support\Collection $results, $page) {
	// do something with results: write to file, etc...
});

// simple aggregates
all()->sum('price') // 101.23
all()->avg('price') // 53.21

// grouped aggregates
groupBy('type')->sum('price') // ['simple' => 101.23, 'digital' => 45.01]
groupBy(['type', 'other_field'])->sum('price') // you can group 'em however you like
```

#### 3. Tuning
Pass tuning options to (almost) any fetcher method to tune your results
```php
// callback
...->get(function (\Illuminate\Support\Collection $results) {
	// do as you please, don't forget to return
});

// map-reduce
...->get('{n}.Model.field') // will do a \Set::extract($results, '{n}.Model.field')
...->get('{n}.Model.field', '{n}.Model.field2') // will do a \Set::combine($results, '{n}.Model.field', '{n}.Model.field2')
...->get('{n}.Model.field', '{n}.Model.field2', '{n}.Model.groupField') // will do a \Set::combine($results, '{n}.Model.field', '{n}.Model.field2', '{n}.Model.groupField')

// disable tuning
...->get(false) // raw results
```

##### Auto-tuning
1. One record with one field, returns just the value. ```id(4)->field('sku') // 'abc1234'```
2. One record without contains, returns the record sans the model nane. ```where(...)->getFirst() // ['id' => 4, 'sku' => 'abc1234', ..]```
3. Aggregate methods: `count()`, `sum()`, `avg()` returns just the value
4. Two fields and the first one is id, we'll auto combine like a list: `fields('id', 'name') // [1 => 'Joe', 2 => 'Bob']`
5. Neighbors will treat each of `'prev'` and `'next'` as its own result and will be subject to the above scenarios: `neighbors(4)->field('id')->get // ['prev' => 3, 'next' => 5]`
