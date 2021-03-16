---
layout: post
title: Laravel Notes
date: 2018-08-28 22:41 +0800
tags:
  - Development
---

These are hard to remember.

### About PHP

`class` and `function`

```php
class Test {
  public static $var = 'var';

  public function func(...$array) {
    foreach ($array as $item)
      echo "item: $item\n";
    
    $format = "item: %s\n";
    
    $println = function($item) use ($format) {
      echo sprintf($format, $item);
    }
      
    return $this::$var;
  }
}
```

### File

To upload multiple files, put each file into parameter name called `uploads[i]` (not `uploads`)

```php
foreach ($request->file('uploads') as $file) {
  $dir = 'uploads';
  $name = $file->getClientOriginalName();
  $disk = 'public'; // see: config/filesystems.php
  $file->storeAs($dir, $name, $disk);
}
```

### Eloquent

Selecting over many to many pivot:

```php
User::has('items', '>', '3'); // items count > 3
User::whereHas('items', function ($query) {
  return $query->whereIn('user_item.item_id', [1, 2, 3]);
});
```

Model attribute casting, this is useful when you need to store an array in the database:

```php
class TheModel extends Model
{
    protected $table = 'the_table_name';
    protected $fillable = ['data'];
    protected $casts = ['data' => 'array'];
}
```

Soft deletes.

```php
class TheModel extends Model
{
    use SoftDeletes;
}

// migration
Schema::table('the_table', function (Blueprint $table) {
    $table->softDeletes();
});
```

