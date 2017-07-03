# fluid-query-builder-cakephp
Ever wanted to write queries in CakePHP fluidly? Does `Model->id(1)->get()` look better than `Model->find('first', ['conditions' => ['Model.id' => 1]])`?

## Three parts:
1. Building
2. Fetching
3. Tuning

### 1. Building: common scenarios
|Method|Does|Notes|
|---|---|---|
|`id(1)`|Shortcut to `where()`||
|`fields('sku')`|certain fields|Is stackable|
|<ul><li>`where(['sku' => '001-0003'])`</li><li>`where('sku = "001-0003"')`</li></ul>|conditions|<ul><li>Array or string</li><li>is stackable</li></ul>|
|`contain(['OtherModel'])`|contain||
|<ul><li>`orderBy('id') // id ASC`</li><li>`orderBy('id DESC')`</li><li>`orderBy(['id' => 'DESC'])`</li><li>`orderBy(['id' => 'DESC', 'price' => 'ASC']) // id DESC, price ASC`</li><li>`orderBy(['id' => 'DESC'])->orderBy('price') // same as previous`</li></ul>|order|<ul><li>Array or string</li><li>is stackable</li></ul>|
|`limit(2)->offset(4)`|limit && offset|may be used separately|
|`page(2, 50)`|page|page 2, page size 50|
|<ul><li>`random()->getFirst() // gets one`</li><li>`random()->limit(5)->get() // gets 5`</li></ul>|random|Shortcut for `orderBy('rand()')`|
|`neighbors(1, 'id')`|neighbors|Pro tip: combine with `where()` to step through search results|

### 2. Fetching
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

### 3. Tuning
Pass tuning options to (almost) any fetcher method to tune your results
#### callback
```php
...->get(function (\Illuminate\Support\Collection $results) {
	// do as you please, don't forget to return
});
```

#### map-reduce
```php
...->get('{n}.Model.field') // will do a \Set::extract($results, '{n}.Model.field')
...->get('{n}.Model.field', '{n}.Model.field2') // will do a \Set::combine($results, '{n}.Model.field', '{n}.Model.field2')
...->get('{n}.Model.field', '{n}.Model.field2', '{n}.Model.groupField') // will do a \Set::combine($results, '{n}.Model.field', '{n}.Model.field2', '{n}.Model.groupField')
```

#### Auto-tuning
1. Getting one record with one field, will just return the field value. ```id(4)->field('sku') // 'abc1234'```
2. Getting one record without contains, will return the record without the model nane. ```where(...)->getFirst() // ['id' => 4, 'sku' => 'abc1234', ..]```
3. Using aggregate methods: `count()`, `sum()`, `avg()` will return just the value
4. Requesting 2 fields and the first one is id, we will auto combine like a list: `fields('id', 'name') // [1 => 'Joe', 2 => 'Bob']`
5. Requesting neighbors: `neighbors()` will treat each of 'prev' and 'next' as its own result and will be subject to the above scenarios

#### disable tuning
```php
...->get(false) // give raw results
```
