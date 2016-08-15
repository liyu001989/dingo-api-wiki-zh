The commands that are available will vary depending on which framework you're using. The following table shows the commands available and what framework they're available in.

|            | Laravel | Lumen |
|------------|:-------:|:-----:|
| api:routes | ✔      |       |
| api:cache  | ✔      |       |
| api:docs   | ✔      |   ✔  |

#### `api:routes`

**Only available in Laravel 5.1+**

This command will generate a table list of your API routes. This command behaves in the exact same way as the `route:list` command available in Laravel. As well as the standard filtering available you also have the following filters: `--versions` and `--scopes`.

##### Examples

```
$ php artisan api:routes
```

```
$ php artisan api:routes --versions v1
```

```
$ php artisan api:routes --scopes read_user_data --scopes write_user_data
```

#### `api:cache`

**Only available in Laravel 5.1+**

This command will cache your API routes alongside the main application routes. When run this command will also run the `route:cache` command automatically, so do not run that command after running this command.

Your routes should either appear in the main `app/Http/routes.php` file or be included within that file for caching to take affect.

##### Examples

```
$ php artisan api:cache
```

#### `api:docs`

**Available in both Laravel 5.1+ and Lumen 5.1+**

This command will generate documentation from your annotated controllers into a standards compliant API Blueprint 1A document. For more on how to annotate your controllers see the [API Blueprint Documentation](https://github.com/dingo/api/wiki/API-Blueprint-Documentation) chapter.

By default the command will output the generated documentation to `stdout` where you can pipe it to a file or push it to a server.

##### Examples

```
$ php artisan api:docs --name Example --use-version v2
```

Output directly to a file with `--output-file`.

```
$ php artisan api:docs --name Example --use-version v2 --output-file /path/to/documentation.md
```

To avoid defining the name and version manually, you can configure defaults in your configuration file or environment file. For more see the [configuration](https://github.com/dingo/api/wiki/Configuration) chapter.

[← API Blueprint Documentation](https://github.com/dingo/api/wiki/API-Blueprint-Documentation)
