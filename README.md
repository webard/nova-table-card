# Nova Table Card

## Simple Nova Card for Displaying Tables

**NOTE**: This has been forked from [whitespacecode/nova-custom-table-card](https://github.com/m-a-k-o/nova-custom-table-card), see [PR #2](https://github.com/whitespacecode/nova-table-card/pull/2). This fork will be deleted from Packagist once the PR is merged.

Simple card table with data of you choice.

It can be useful as latest order list or latest posts, ...

![Nova Table Card](https://github.com/Whitespacecode/nova-table-card/blob/master/example.png)

## Requirements

- `php: >=8.1`
- `laravel/nova: ^4.0`

## Installation

You can install the package in to a Laravel app that uses [Nova](https://nova.laravel.com) via composer:

```bash
composer require webard/nova-table-card
```

You must register the card with NovaServiceProvider.

```php
// in app/Providers/NovaServiceProvder.php

// ...
public function cards()
{
    return [
        // ...

        // all the parameters are required excelpt title
        new \Whitespacecode\TableCard\TableCard(
            array $header, array $data, string $title, array $viewAll
        ),
    ];
}
```

Example of use:

```php
use Whitespacecode\TableCard\TableCard;
use Whitespacecode\TableCard\Table\Cell;
use Whitespacecode\TableCard\Table\Row;

// ...
public function cards()
{
    return [
        // ...

        // all the parameters are required except title and/or viewLink
        (new TableCard)
            ->header([
                Cell::make('Order Number'),
                // Set sortable to true in a header Cell to allow its column's sorting
                (Cell::make('Price'))->sortable(true)->class('text-right'),
            ])
            ->data([
                (Row::make(
                    Cell::make('2018091001'),
                    (Cell::make('20.50'))->class('text-right')->id('price-2')
                ))->viewLink('/resources/orders/1'), //Add viewLink to show clickable eye
                (Row::make(
                    Cell::make('2018091002'),
                    (Cell::make('201.25'))->class('text-right')->id('price-2')
                )),
            ])
            ->title('Orders')
            ->viewAll(['label' => 'View All', 'link' => '/resources/orders']),
    ];
}
```

or:

You can create your own class which will extend \Whitespacecode\TableCard\TableCard in Nova/Cards directory on example.

In this separate class you are able to fetch data from models in nice clean way.

```php
<?php

namespace App\Nova\Cards;

use Whitespacecode\TableCard\TableCard;
use Whitespacecode\TableCard\Table\Cell;
use Whitespacecode\TableCard\Table\Row;

use App\Models\Order;

class LatestOrders extends TableCard
{
    public function __construct()
    {
        $header = collect(['Date', 'Order Number', 'Status', 'Price', 'Name']);

        $this->title('Latest Orders');
        $this->viewAll([
            'label' => 'View All', //Default value `View All`
            'link' => '/resources/orders', //Required field
            'position' => 'bottom' //Default value `top`
            'style' => 'button' //Default value `link`
        ]);

        // $orders = Order::take(10)->latest->get();
        // Data from you model
        $orders = collect([
            ['date' => '2018-12-01', 'order_number' => '2018120101', 'status' => 'Ordered', 'price' => '20.55', 'name' => 'John Doe'],
            ['date' => '2018-12-01', 'order_number' => '2018120101', 'status' => 'Ordered', 'price' => '20.55', 'name' => 'John Doe'],
            ['date' => '2018-12-01', 'order_number' => '2018120101', 'status' => 'Ordered', 'price' => '20.55', 'name' => 'John Doe'],
            ['date' => '2018-12-01', 'order_number' => '2018120101', 'status' => 'Ordered', 'price' => '20.55', 'name' => 'John Doe'],
            ['date' => '2018-12-01', 'order_number' => '2018120101', 'status' => 'Ordered', 'price' => '20.55', 'name' => 'John Doe'],
            ['date' => '2018-12-01', 'order_number' => '2018120101', 'status' => 'Ordered', 'price' => '20.55', 'name' => 'John Doe'],
        ]);

        $this->header($header->map(function($value) {
            // Make the Status column sortable
            return ($value === 'Status') ?
                (Cell::make($value))->sortable(true) :
                Cell::make($value);
        })->toArray());

        $this->data($orders->map(function($order) {
            return Row::make(
                Cell::make($order['date']),
                Cell::make($order['order_number']),
                // Instead of alphabetically ordering the status, set a sortableData value for better representation
                (Cell::make($order['status'])->sortableData($this->getStatusSortableData($order['status']))),
                Cell::make($order['price']),
                Cell::make($order['name'])
            );
        })->toArray());

        //Possible style configuration
        // $this->style = 'tight';

    }

    private function getStatusSortableData (string $status) : int
    {
        switch ($status) {
            case 'Ordered':
                return 1;
            default:
                return 0.
        }
    }
}
```

Then simply include your custom class like a normal card within your resource

```php
use App\Nova\Cards\LatestOrders;

protected function cards()
{
    return [
        ......
        new LatestOrders,
     ];
 }
```

#### Note:

If you don't specify viewLink() on a row `Row::make()->viewLink()`, show icon will not be visible.

#### Additional Fields in viewAll

You can also show a viewAll on the table with `$this->viewAll()`

- **label (optional)**: By default, it is set to 'View All'.
- **position (optional)**: By default, it is set to 'top'. You can change it to 'bottom' if needed.
- **style (optional)**: DThe default style is a 'link'. Alternatively, you can set it to 'button' for a button-style appearance.

```php
$this->viewAll([
    'label' => '__('Latest Orders')',
    'link' => '/resources/orders', //URL to navigate when the link is clicked
    'position' => 'bottom', //(Possible values `top` - `bottom`)
    'style' => 'button', //(Possible values `link` - `button`)
]);
```

## Table Style Customization

To show more data on your table, you can use the "tight" table style option designed to increase the visual density of your table rows.

```php
use Whitespacecode\TableCard\TableCard;

protected function cards()
{
    return [
        ...
        TableCard::make(
            ...
        )->style('tight'),
     ];
 }
```

Or override the `$style` property on your custom class:

```php
$this->style = 'tight';
```

## Using the pagination

The pagination accepts a default `Illuminate\Pagination\LengthAwarePaginator`

When getting your data just use the default `->paginate()` and pass your data to the paginator.<br>
Everything else stays the same.

```php
class LatestOrders extends TableCard
{
    public function __construct()
    {
        $orders = Orders::paginate(5);

        //You want to start by showing the latest result? Get them by latest
        //$orders = Orders::latest()->paginate(5);

        $this->paginator($orders);
    }
}
```

![Nova Table Card](https://github.com/Whitespacecode/nova-table-card/blob/master/examplePaginate.png)
