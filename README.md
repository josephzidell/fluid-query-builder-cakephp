# fluid-query-builder-cakephp
Ever wanted to write queries in CakePHP fluidly? Does `Model->id(1)->get()` look better than `Model->find('first', ['conditions' => ['Model.id' => 1]])`?

## Three parts:
1. Building
2. Fetching
3. Tuning

### 1. Building: common scenarios
|Method|Does|Is stackable?|
|---|---|---|
|`id(1)`|Shortcut to `where()`||
|`fields('sku')`|certain fields|Yes|

// where conditions
Model::where(['sku' => '001-0003'])->getFirst() // single record
Model::where('sku = "001-0003"')->get() // string conditions work too
Model::where('sku = "001-0003"')->where(['price >' => 100])->get() // yep, it's stackable

// contain
Model::id(1)->contain(['OtherModel'])->get() // single record with associated OtherModel

// order
Model::orderBy('id') // id ASC
Model::orderBy('id DESC') // id DESC
Model::orderBy(['id' => 'DESC']) // id DESC
Model::orderBy(['id' => 'DESC', 'price' => 'ASC']) // id DESC, price ASC
Model::orderBy(['id' => 'DESC'])->orderBy('price') // stackable: id DESC, price ASC

// limit && offset
Model::limit(2)->offset(4)->get()

// page
Model::page(2, 50)->get() // page 2, page size 50

// random
Model::random()->getFirst() // gets one
Model::random()->limit(5)->get() // gets 5 random records

// neighbors
Model::neighbors(1, 'id') // ['prev' => ..., 'next' => ...]
Model::where($mySearchCriteria)->neighbors(1, 'id') // can navigate through search results on view page
```

### 2. Fetching
```php
// list
Model::fields('id', 'sku')->get() // [1 => 'First record', 2 => 'Second record']

// count
Model::where(['sku LIKE' => '001-%'])->count() // 5

// chunking
Model::where(['to_export' => true])->chunk(100, function (\Illuminate\Support\Collection $results, $page) {
	// do something with results: write to file, etc...
});

// simple aggregates
Model::all()->sum('price') // 101.23
Model::all()->avg('price') // 53.21

// grouped aggregates
Model::groupBy('type')->sum('price') // ['simple' => 101.23, 'digital' => 45.01]
Model::groupBy(['type', 'other_field'])->sum('price') // you can group 'em however you like
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
