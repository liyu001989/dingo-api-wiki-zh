The commands that are available will vary depending on which framework you're using. The following table shows the commands available and what framework they're available in.

可用的命令取决于你正在使用的框架。下面这个表显示了框架和命令之间的可用关系。

|            | Laravel | Lumen |
|------------|:-------:|:-----:|
| api:routes | ✔      |       |
| api:cache  | ✔      |       |
| api:docs   | ✔      |   ✔  |

#### `api:routes`

**Only available in Laravel 5.1+**

**只有 Laravel 5.1+ 可用**

This command will generate a table list of your API routes. This command behaves in the exact same way as the `route:list` command available in Laravel. As well as the standard filtering available you also have the following filters: `--versions` and `--scopes`.

这个命令将生成你的 API 路由列表。这个命令的效果类似 Laravel 中的 `route:list` 命令。除了标准的使用方法，你还可以使用以下的过滤器：`--versions` 和 `--scopes`.

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

**只有 Laravel 5.1+ 可用**


This command will cache your API routes alongside the main application routes. When run this command will also run the `route:cache` command automatically, so do not run that command after running this command.

这个命令将缓存你的 API 路由，和你主要应用的路由一起。当执行这个命令的时候会自动执行 `route:cache`  命令。所以执行完这个命令后就不要在执行 `route:cache` 了。

Your routes should either appear in the main `app/Http/routes.php` file or be included within that file for caching to take affect.

你的路由需要出现在你的 `app/Http/routes.php` 文件中或者被这个文件引入的文件中，从而让缓存个生效。

##### Examples

```
$ php artisan api:cache
```

#### `api:docs`

**Available in both Laravel 5.1+ and Lumen 5.1+**

**只有 Laravel 5.1+ 和 Lumen 5.1+ 可用**


This command will generate documentation from your annotated controllers into a standards compliant API Blueprint 1A document. For more on how to annotate your controllers see the [API Blueprint Documentation](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/API-Blueprint-Documentation) chapter.

这个命令将从你的控制器注释中生成文档到一个符合标准的api文档中。更多的如何注释你的控制器请看 [API Blueprint Documentation](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/API-Blueprint-Documentation) 章节。

By default the command will output the generated documentation to `stdout` where you can pipe it to a file or push it to a server.

默认的，这个命令将把文档输出到 `stdout` 中，你可以使用管道将这个文件存储到一个文件中或者推送到服务器上。

##### Examples

```
$ php artisan api:docs --name Example --use-version v2
```

Output directly to a file with `--output-file`.

文件的输出目录 使用 `--output-file`。

```
$ php artisan api:docs --name Example --use-version v2 --output-file /path/to/documentation.md
```

To avoid defining the name and version manually, you can configure defaults in your configuration file or environment file. For more see the [configuration](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Configuration.md) chapter.

为了避免手动定义名字和版本，你可以自定义配置到你的配置文件或者环境文件中。详情请看 [configuration](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Configuration.md) 章节.

[← API Blueprint Documentation](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/API-Blueprint-Documentation.md)
