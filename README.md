![Filament Archivable](https://github.com/statikbe/filament-archivable/raw/main/assets/screen-header.png)
# Filament plugin to archive, unarchive and filter records

[![Latest Version on Packagist](https://img.shields.io/packagist/v/statikbe/filament-archivable.svg?style=flat-square)](https://packagist.org/packages/statikbe/filament-archivable)
[![GitHub Tests Action Status](https://img.shields.io/github/actions/workflow/status/statikbe/filament-archivable/run-tests.yml?branch=main&label=tests&style=flat-square)](https://github.com/statikbe/filament-archivable/actions?query=workflow%3Arun-tests+branch%3Amain)
[![GitHub Code Style Action Status](https://img.shields.io/github/actions/workflow/status/statikbe/filament-archivable/fix-php-code-style-issues.yml?branch=main&label=code%20style&style=flat-square)](https://github.com/statikbe/filament-archivable/actions?query=workflow%3A"Fix+PHP+code+style+issues"+branch%3Amain)
[![Total Downloads](https://img.shields.io/packagist/dt/statikbe/filament-archivable.svg?style=flat-square)](https://packagist.org/packages/statikbe/filament-archivable)

Filament plugin for archiving and unarchiving table records (eloquent models) based on the [Laravel Archivable package by Joe Butcher](https://github.com/joelbutcher/laravel-archivable).

This filament plugin adds an [ArchiveAction](#archive-and-unarchive-table-actions), an [UnArchiveAction](#archive-and-unarchive-table-actions) and an [ArchivedFilter](#filtering) to your resource tables. It's also possible to [add custom classes to archived table rows](#add-custom-classes-to-archived-rows). The same actions can be used as [page-actions](#archive-and-unarchive-page-actions) on view/edit pages to archive and unarchive your records.

## Requirements

- PHP ^8.3
- Laravel ^11.0 || ^12.0 || ^13.0
- Filament ^4.0 || ^5.0
- Laravel Archivable ^1.4 (installed with this plugin)

> Looking for Filament v3? Use the [`okeonline/filament-archivable`](https://github.com/okeonline/filament-archivable) package this plugin was forked from.

> The Filament documentation links below point to the v4 docs. If you are on Filament v5, the same pages are available under [`/docs/5.x/`](https://filamentphp.com/docs/5.x).

## Installation

You can install the package via composer:

```bash
composer require statikbe/filament-archivable
```

Then, add the plugin to your `PanelProvider`:

```php
class AppPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            // ...
            ->plugin(\Statik\FilamentArchivable\FilamentArchivablePlugin::make());
    }
}
```

This filament plugin automatically installs the [Laravel Archivable package by Joe Butcher](https://github.com/joelbutcher/laravel-archivable).

Follow his installation instructions, which -in short- instructs:

1) Add an `archived_at` column to your table
2) Use the `Archivable`-trait on your model

## Usage

### Archive and UnArchive table actions
As soon as the `Archivable`-trait from the [Laravel Archivable package](#installation) is added to the model, it is possible to add the following actions to the corresponding resource table:

```php
use Statik\FilamentArchivable\Actions\ArchiveAction;
use Statik\FilamentArchivable\Actions\UnArchiveAction;
// ...

class UserResource extends Resource
{
    // ...
    public static function table(Table $table): Table
    {
        return $table
            // ...
            ->recordActions([
                ArchiveAction::make(),
                UnArchiveAction::make(),
            ]);
    }
}
```

It will show the `ArchiveAction` on records that aren't archived, and will show the `UnArchiveAction` on those which are currently archived:

![Actions](https://github.com/statikbe/filament-archivable/raw/main/assets/screen-actions.png)

> You should add **both** actions to the same table. The action itself will determine if it should be shown on the record.

The actions are normal record actions, similar to the [Delete](https://filamentphp.com/docs/4.x/actions/delete) and [Restore](https://filamentphp.com/docs/4.x/actions/restore) actions of FilamentPHP. You can add all features that are described in the [FilamentPHP Actions Documentation](https://filamentphp.com/docs/4.x/actions/overview), like:

- `hiddenLabel()`
- `tooltip()`
- `disabled()`
- `icon()`
- ... etc.

```php
ArchiveAction::make()
    ->hiddenLabel()
    ->tooltip('Archive'),
```

The actions call the `$model->archive()` and `$model->unArchive()` methods that are provided by the [Laravel Archivable package](https://github.com/joelbutcher/laravel-archivable?tab=readme-ov-file#extensions).

### Archive and UnArchive page actions
The same `ArchiveAction` and `UnArchiveAction` classes can be used as header actions on resource **pages**, e.g. an edit page:

```php
// in e.g. Filament/Resources/PostResource/Pages/EditPost.php
use Statik\FilamentArchivable\Actions\ArchiveAction;
use Statik\FilamentArchivable\Actions\UnArchiveAction;

// ...
protected function getHeaderActions(): array
{
    return [
        ArchiveAction::make(),
        UnArchiveAction::make(),
    ];
}
```

It will show the `ArchiveAction` on records that aren't archived, and will show the `UnArchiveAction` on those which are currently archived.

> You should add **both** actions to the same page. The action itself will determine if it should be shown on the record. Check [this](#ability-to-viewedit-delete-archived-records) if you want to edit (and unarchive) records that are being archived.

### Filtering

It is also possible to add an `ArchivedFilter` to the resource table, which adds three filtering options:
- Show only unarchived records
- Show only archived records
- Show both

> By default, an unfiltered table will only show the unarchived records, as the Laravel Archivable package comes with a default global scope to query records with `archived_at IS NULL`

You can add the filter by adding `ArchivedFilter` to your array of resource table filters:

```php
use Statik\FilamentArchivable\Tables\Filters\ArchivedFilter;

public static function table(Table $table): Table
{
    return $table
        // ...
        ->filters([
            ArchivedFilter::make(),
        ]);
}
```

![Filters](https://github.com/statikbe/filament-archivable/raw/main/assets/screen-filters.png)

The `ArchivedFilter` extends Filament's `TernaryFilter`, so check the [Ternary Filter Documentation of Filament](https://filamentphp.com/docs/4.x/tables/filters/ternary) to customize the filter.

#### Ability to view/edit/delete archived records
If you want to be able to view/edit/delete archived records, you should disable the global `ArchivableScope::class` on your resource `getEloquentQuery()` method:

```php
// in your resource:
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\SoftDeletingScope;
use LaravelArchivable\Scopes\ArchivableScope;

public static function getEloquentQuery(): Builder
{
    return parent::getEloquentQuery()
        ->withoutGlobalScopes([
            SoftDeletingScope::class, // only if soft deleting is also active, otherwise it can be omitted
            ArchivableScope::class,
        ]);
}
```
See [Disabling global scopes on Filament](https://filamentphp.com/docs/4.x/resources#disabling-global-scopes) for more information about the default resource query.

### Add custom classes to archived rows

The plugin automatically adds CSS/Tailwind classes to **table rows** of records that are archived. By default it applies `opacity-25` to any record using the `Archivable` trait.

You can customize these classes when registering the plugin in your `PanelProvider`:

```php
class AppPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            // ...
            ->plugin(
                \Statik\FilamentArchivable\FilamentArchivablePlugin::make()
                    ->archivedRecordClasses(['opacity-25', 'grayscale'])
            );
    }
}
```

![Custom classes](https://github.com/statikbe/filament-archivable/raw/main/assets/screen-classes.png)

## Supported languages

- en - English
- fr - French
- de - German
- nl - Dutch

You can publish and change the language files by running:

```bash
php artisan vendor:publish --tag=filament-archivable-translations
```

## Testing

```bash
composer test
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [Rudi van Zandwijk](https://github.com/rvzug)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
