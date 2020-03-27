# Параметры командной строки

<!--introduced_in=v5.9.1-->
<!--type=misc-->

Node.js поставляется с различными вариантами CLI. Эти параметры предоставляют встроенную отладку, несколько способов запуска сценариев и другие полезные параметры времени выполнения.

To view this documentation as a manual page in a terminal, run `man node`.

## Краткий обзор

`node [options] [V8 options] [script.js | -e "script" | -] [--] [arguments]`

`node inspect [script.js | -e "script" | <host>:<port>] …`

`узел --v8-опций`

Выполнить без аргументов, чтобы запустить [REPL](repl.html).

_For more info about `node inspect`, please see the [debugger](debugger.html) documentation._

## Опции
<!-- YAML
changes:
  - version: v10.12.0
    pr-url: https://github.com/nodejs/node/pull/23020
    description: Underscores instead of dashes are now allowed for
                 Node.js options as well, in addition to V8 options.
-->

All options, including V8 options, allow words to be separated by both dashes (`-`) or underscores (`_`).

For example, `--pending-deprecation` is equivalent to `--pending_deprecation`.

### `-`
<!-- YAML
added: v8.0.0
-->

Alias for stdin. Analogous to the use of `-` in other command line utilities, meaning that the script will be read from stdin, and the rest of the options are passed to that script.

### `--`
<!-- YAML
added: v6.11.0
-->

Укажите параметры конца узла. Передайте остальные аргументы сценарию. Если до этого ранее не указывалось имя сценария или сценарий eval/print, в таком случае следующий аргумент будет использован как имя сценария.

### `--abort-on-uncaught-exception`
<!-- YAML
added: v0.10.8
-->

Aborting instead of exiting causes a core file to be generated for post-mortem analysis using a debugger (such as `lldb`, `gdb`, and `mdb`).

If this flag is passed, the behavior can still be set to not abort through [`process.setUncaughtExceptionCaptureCallback()`][] (and through usage of the `domain` module that uses it).

### `--completion-bash`
<!-- YAML
added: v10.12.0
-->

Print source-able bash completion script for Node.js.

```console
$ node --completion-bash > node_bash_completion
$ source node_bash_completion
```

### `--cpu-prof`
<!-- YAML
added: v12.0.0
-->

> Стабильность: 1 - экспериментальный

Starts the V8 CPU profiler on start up, and writes the CPU profile to disk before exit.

If `--cpu-prof-dir` is not specified, the generated profile will be placed in the current working directory.

If `--cpu-prof-name` is not specified, the generated profile will be named `CPU.${yyyymmdd}.${hhmmss}.${pid}.${tid}.${seq}.cpuprofile`.

```console
$ node --cpu-prof index.js
$ ls *.cpuprofile
CPU.20190409.202950.15293.0.0.cpuprofile
```

### `--cpu-prof-dir`
<!-- YAML
added: v12.0.0
-->

> Стабильность: 1 - экспериментальный

Specify the directory where the CPU profiles generated by `--cpu-prof` will be placed.

### `--cpu-prof-interval`
<!-- YAML
added: v12.2.0
-->

> Стабильность: 1 - экспериментальный

Specify the sampling interval in microseconds for the CPU profiles generated by `--cpu-prof`. The default is 1000 microseconds.

### `--cpu-prof-name`
<!-- YAML
added: v12.0.0
-->

> Стабильность: 1 - экспериментальный

Specify the file name of the CPU profile generated by `--cpu-prof`.

### `--disallow-code-generation-from-strings`
<!-- YAML
added: v9.8.0
-->

Make built-in language features like `eval` and `new Function` that generate code from strings throw an exception instead. This does not affect the Node.js `vm` module.

### `--enable-fips`
<!-- YAML
added: v6.0.0
-->

Включите FIPS-совместимое шифрование при запуске. (Requires Node.js to be built with `./configure --openssl-fips`.)

### `--enable-source-maps`
<!-- YAML
added: v12.12.0
-->

> Стабильность: 1 - экспериментальный

Enable experimental Source Map V3 support for stack traces.

Currently, overriding `Error.prepareStackTrace` is ignored when the `--enable-source-maps` flag is set.

### `--experimental-import-meta-resolve`
<!-- YAML
added: v13.9.0
-->

Enable experimental `import.meta.resolve()` support.

### `--experimental-json-modules`
<!-- YAML
added: v12.9.0
-->

Enable experimental JSON support for the ES Module loader.

### `--experimental-modules`
<!-- YAML
added: v8.5.0
-->

Enable latest experimental modules features (deprecated).

### `--experimental-policy`
<!-- YAML
added: v11.8.0
-->

Use the specified file as a security policy.

### `--experimental-repl-await`
<!-- YAML
added: v10.0.0
-->

Enable experimental top-level `await` keyword support in REPL.

### `--experimental-report`
<!-- YAML
added: v11.8.0
-->

Enable experimental diagnostic report feature.

### `--experimental-specifier-resolution=mode`
<!-- YAML
added: v13.4.0
-->

Sets the resolution algorithm for resolving ES module specifiers. Valid options are `explicit` and `node`.

The default is `explicit`, which requires providing the full path to a module. The `node` mode will enable support for optional file extensions and the ability to import a directory that has an index file.

Please see [customizing ESM specifier resolution](esm.html#esm_customizing_esm_specifier_resolution_algorithm) for example usage.

### `--experimental-vm-modules`
<!-- YAML
added: v9.6.0
-->

Enable experimental ES Module support in the `vm` module.

### `--experimental-wasi-unstable-preview1`
<!-- YAML
added: v13.3.0
changes:
  - version: v13.6.0
    pr-url: https://github.com/nodejs/node/pull/30980
    description: changed from `--experimental-wasi-unstable-preview0` to
                 `--experimental-wasi-unstable-preview1`
-->

Enable experimental WebAssembly System Interface (WASI) support.

### `--experimental-wasm-modules`
<!-- YAML
added: v12.3.0
-->

Enable experimental WebAssembly module support.

### `--force-fips`
<!-- YAML
added: v6.0.0
-->

Принудительно включается FIPS-совместимое шифрование при запуске. (Cannot be disabled from script code.) (Same requirements as `--enable-fips`.)

### `--frozen-intrinsics`
<!-- YAML
added: v11.12.0
-->

> Стабильность: 1 - экспериментальный

Enable experimental frozen intrinsics like `Array` and `Object`.

Support is currently only provided for the root context and no guarantees are currently provided that `global.Array` is indeed the default intrinsic reference. Code may break under this flag.

`--require` runs prior to freezing intrinsics in order to allow polyfills to be added.

### `--heapsnapshot-signal=signal`
<!-- YAML
added: v12.0.0
-->

Enables a signal handler that causes the Node.js process to write a heap dump when the specified signal is received. `signal` must be a valid signal name. Disabled by default.

```console
$ node --heapsnapshot-signal=SIGUSR2 index.js &
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
node         1  5.5  6.1 787252 247004 ?       Ssl  16:43   0:02 node --heapsnapshot-signal=SIGUSR2 index.js
$ kill -USR2 1
$ ls
Heap.20190718.133405.15554.0.001.heapsnapshot
```

### `--heap-prof`
<!-- YAML
added: v12.4.0
-->

> Стабильность: 1 - экспериментальный

Starts the V8 heap profiler on start up, and writes the heap profile to disk before exit.

If `--heap-prof-dir` is not specified, the generated profile will be placed in the current working directory.

If `--heap-prof-name` is not specified, the generated profile will be named `Heap.${yyyymmdd}.${hhmmss}.${pid}.${tid}.${seq}.heapprofile`.

```console
$ node --heap-prof index.js
$ ls *.heapprofile
Heap.20190409.202950.15293.0.001.heapprofile
```

### `--heap-prof-dir`
<!-- YAML
added: v12.4.0
-->

> Стабильность: 1 - экспериментальный

Specify the directory where the heap profiles generated by `--heap-prof` will be placed.

### `--heap-prof-interval`
<!-- YAML
added: v12.4.0
-->

> Стабильность: 1 - экспериментальный

Specify the average sampling interval in bytes for the heap profiles generated by `--heap-prof`. The default is 512 * 1024 bytes.

### `--heap-prof-name`
<!-- YAML
added: v12.4.0
-->

> Стабильность: 1 - экспериментальный

Specify the file name of the heap profile generated by `--heap-prof`.

### `--icu-data-dir=file`
<!-- YAML
added: v0.11.15
-->

Задается путь загрузки данных ICU. (Перезаписывает `NODE_ICU_DATA`.)

### `--input-type=type`
<!-- YAML
added: v12.0.0
-->

This configures Node.js to interpret string input as CommonJS or as an ES module. String input is input via `--eval`, `--print`, or `STDIN`.

Valid values are `"commonjs"` and `"module"`. The default is `"commonjs"`.

### `--inspect-brk[=[host:]port]`
<!-- YAML
added: v7.6.0
-->

Activate inspector on `host:port` and break at start of user script. Default `host:port` is `127.0.0.1:9229`.

### `--inspect-port=[host:]port`
<!-- YAML
added: v7.6.0
-->

Set the `host:port` to be used when the inspector is activated. Useful when activating the inspector by sending the `SIGUSR1` signal.

Default host is `127.0.0.1`.

See the [security warning](#inspector_security) below regarding the `host` parameter usage.

### `--inspect[=[host:]port]`
<!-- YAML
added: v6.3.0
-->

Activate inspector on `host:port`. Default is `127.0.0.1:9229`.

V8 inspector integration allows tools such as Chrome DevTools and IDEs to debug and profile Node.js instances. The tools attach to Node.js instances via a tcp port and communicate using the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/).

<a id="inspector_security"></a>

#### Warning: binding inspector to a public IP:port combination is insecure

Binding the inspector to a public IP (including `0.0.0.0`) with an open port is insecure, as it allows external hosts to connect to the inspector and perform a [remote code execution](https://www.owasp.org/index.php/Code_Injection) attack.

If specifying a host, make sure that either:

* The host is not accessible from public networks.
* A firewall disallows unwanted connections on the port.

**More specifically, `--inspect=0.0.0.0` is insecure if the port (`9229` by default) is not firewall-protected.**

See the [debugging security implications](https://nodejs.org/en/docs/guides/debugging-getting-started/#security-implications) section for more information.

### `--inspect-publish-uid=stderr,http`

Specify ways of the inspector web socket url exposure.

By default inspector websocket url is available in stderr and under `/json/list` endpoint on `http://host:port/json/list`.

### `--experimental-loader=module`
<!-- YAML
added: v9.0.0
-->

Specify the `module` of a custom [experimental ECMAScript Module loader](esm.html#esm_experimental_loaders). `module` may be either a path to a file, or an ECMAScript Module name.

### `--insecure-http-parser`
<!-- YAML
added: v13.4.0
-->

Use an insecure HTTP parser that accepts invalid HTTP headers. This may allow interoperability with non-conformant HTTP implementations. It may also allow request smuggling and other HTTP attacks that rely on invalid headers being accepted. Avoid using this option.

### `--max-http-header-size=size`
<!-- YAML
added: v11.6.0
-->

Specify the maximum size, in bytes, of HTTP headers. Defaults to 8KB.

### `--napi-modules`
<!-- YAML
added: v7.10.0
-->

This option is a no-op. It is kept for compatibility.

### `--no-deprecation`
<!-- YAML
added: v0.8.0
-->

Тихое предупреждение об устаревании.

### `--no-force-async-hooks-checks`
<!-- YAML
added: v9.0.0
-->

Disables runtime checks for `async_hooks`. These will still be enabled dynamically when `async_hooks` is enabled.

### `--no-warnings`
<!-- YAML
added: v6.0.0
-->

Делает все процессные уведомления тихими (включая устаревшие версии).

### `--force-context-aware`
<!-- YAML
added: v12.12.0
-->

Disable loading native addons that are not [context-aware](addons.html#addons_context_aware_addons).

### `--openssl-config=file`
<!-- YAML
added: v6.9.0
-->

Загрузка файла конфигурации OpenSSL при запуске. Среди прочего это может использоваться для включения FIPS-совместимого шифрования, если Node.js создан с `./configure --openssl-fips`.

### `--pending-deprecation`
<!-- YAML
added: v8.0.0
-->

Emit pending deprecation warnings.

Pending deprecations are generally identical to a runtime deprecation with the notable exception that they are turned *off* by default and will not be emitted unless either the `--pending-deprecation` command line flag, or the `NODE_PENDING_DEPRECATION=1` environment variable, is set. Pending deprecations are used to provide a kind of selective "early warning" mechanism that developers may leverage to detect deprecated API usage.

### `--policy-integrity=sri`
<!-- YAML
added: v12.7.0
-->

> Стабильность: 1 - экспериментальный

Instructs Node.js to error prior to running any code if the policy does not have the specified integrity. It expects a [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) string as a parameter.

### `--preserve-symlinks`
<!-- YAML
added: v6.3.0
-->

Указывает загрузчику модулей сохранять символические ссылки при разрешении и кэшировании модулей.

По умолчанию, когда Node.Js загружает модуль из маршрута, который символически связан с другим местоположением на диске, Node.Js разыменовывает ссылку и использует текущий «реальный маршрут» дискового модуля как идентификатор и корневой маршрут, чтобы найти другие модули зависимостей. В большинстве случаев это поведение по умолчанию является приемлемым. Однако, как показано в нижеследующем примере, когда используются символически связанные парные зависимости, поведение по умолчанию вызывает исключение, если `moduleA` пытается запросить `moduleB` в качестве парной зависимости:

```text
{appDir}
 ├── app
 │   ├── index.js
 │   └── node_modules
 │       ├── moduleA -> {appDir}/moduleA
 │       └── moduleB
 │           ├── index.js
 │           └── package.json
 └── moduleA
     ├── index.js
     └── package.json
```

Флаг командной строки `--preserve-symlinks` указывает Node.js использовать путь символической ссылки в отличие от реального пути, что позволяет найти символически связанные парные зависимости.

Обратите внимание, что использование `--preserve-symlinks` может иметь другие побочные эффекты. Specifically, symbolically linked *native* modules can fail to load if those are linked from more than one location in the dependency tree (Node.js would see those as two separate modules and would attempt to load the module multiple times, causing an exception to be thrown).

The `--preserve-symlinks` flag does not apply to the main module, which allows `node --preserve-symlinks node_module/.bin/<foo>` to work. To apply the same behavior for the main module, also use `--preserve-symlinks-main`.

### `--preserve-symlinks-main`
<!-- YAML
added: v10.2.0
-->

Instructs the module loader to preserve symbolic links when resolving and caching the main module (`require.main`).

This flag exists so that the main module can be opted-in to the same behavior that `--preserve-symlinks` gives to all other imports; they are separate flags, however, for backward compatibility with older Node.js versions.

`--preserve-symlinks-main` does not imply `--preserve-symlinks`; it is expected that `--preserve-symlinks-main` will be used in addition to `--preserve-symlinks` when it is not desirable to follow symlinks before resolving relative paths.

See `--preserve-symlinks` for more information.

### `--prof`
<!-- YAML
added: v2.0.0
-->

Generate V8 profiler output.

### `--prof-process`
<!-- YAML
added: v5.2.0
-->

Process V8 profiler output generated using the V8 option `--prof`.

### `--redirect-warnings=file`
<!-- YAML
added: v8.0.0
-->

Запись процессных уведомлений в заданный файл вместо печати в stderr. Файл будет создан, если он не существует, и будет добавлен, если он существует. Если при попытке записать предупреждение в файл возникает ошибка, предупреждение вместо этого будет записано в stderr.

### `--report-directory=directory`
<!-- YAML
added: v11.8.0
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: Changed from `--diagnostic-report-directory` to
                 `--report-directory`
-->

Location at which the report will be generated.

### `--report-filename=filename`
<!-- YAML
added: v11.8.0
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-filename` to
                 `--report-filename`
-->

Name of the file to which the report will be written.

### `--report-on-fatalerror`
<!-- YAML
added: v11.8.0
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-on-fatalerror` to
                 `--report-on-fatalerror`
-->

Enables the report to be triggered on fatal errors (internal errors within the Node.js runtime such as out of memory) that lead to termination of the application, if `--experimental-report` is enabled. Useful to inspect various diagnostic data elements such as heap, stack, event loop state, resource consumption etc. to reason about the fatal error.

### `--report-on-signal`
<!-- YAML
added: v11.8.0
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-on-signal` to
                 `--report-on-signal`
-->

Enables report to be generated upon receiving the specified (or predefined) signal to the running Node.js process, if `--experimental-report` is enabled. The signal to trigger the report is specified through `--report-signal`.

### `--report-signal=signal`
<!-- YAML
added: v11.8.0
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-signal` to
                 `--report-signal`
-->

Sets or resets the signal for report generation (not supported on Windows). Default signal is `SIGUSR2`.

### `--report-uncaught-exception`
<!-- YAML
added: v11.8.0
changes:
  - version: v12.0.0
    pr-url: https://github.com/nodejs/node/pull/27312
    description: changed from `--diagnostic-report-uncaught-exception` to
                 `--report-uncaught-exception`
-->

Enables report to be generated on un-caught exceptions, if `--experimental-report` is enabled. Useful when inspecting JavaScript stack in conjunction with native stack and other runtime environment data.

### `--throw-deprecation`
<!-- YAML
added: v0.11.14
-->

Формирование ошибок для устаревших версий.

### `--title=title`
<!-- YAML
added: v10.7.0
-->

Set `process.title` on startup.

### `--tls-cipher-list=list`
<!-- YAML
added: v4.0.0
-->

Задается альтернативный список шифров TLS по умолчанию. Requires Node.js to be built with crypto support (default).

### `--tls-keylog=file`
<!-- YAML
added: v13.2.0
-->

Log TLS key material to a file. The key material is in NSS `SSLKEYLOGFILE` format and can be used by software (such as Wireshark) to decrypt the TLS traffic.

### `--tls-max-v1.2`
<!-- YAML
added: v12.0.0
-->

Set [`tls.DEFAULT_MAX_VERSION`][] to 'TLSv1.2'. Use to disable support for TLSv1.3.

### `--tls-max-v1.3`
<!-- YAML
added: v12.0.0
-->

Set default [`tls.DEFAULT_MAX_VERSION`][] to 'TLSv1.3'. Use to enable support for TLSv1.3.

### `--tls-min-v1.0`
<!-- YAML
added: v12.0.0
-->

Set default [`tls.DEFAULT_MIN_VERSION`][] to 'TLSv1'. Use for compatibility with old TLS clients or servers.

### `--tls-min-v1.1`
<!-- YAML
added: v12.0.0
-->

Set default [`tls.DEFAULT_MIN_VERSION`][] to 'TLSv1.1'. Use for compatibility with old TLS clients or servers.

### `--tls-min-v1.2`
<!-- YAML
added: v12.2.0
-->

Set default [`tls.DEFAULT_MIN_VERSION`][] to 'TLSv1.2'. This is the default for 12.x and later, but the option is supported for compatibility with older Node.js versions.

### `--tls-min-v1.3`
<!-- YAML
added: v12.0.0
-->

Set default [`tls.DEFAULT_MIN_VERSION`][] to 'TLSv1.3'. Use to disable support for TLSv1.2, which is not as secure as TLSv1.3.

### `--trace-deprecation`
<!-- YAML
added: v0.8.0
-->

Печать трассировок стека для устаревших версий.

### `--trace-event-categories`
<!-- YAML
added: v7.7.0
-->

A comma separated list of categories that should be traced when trace event tracing is enabled using `--trace-events-enabled`.

### `--trace-event-file-pattern`
<!-- YAML
added: v9.8.0
-->

Template string specifying the filepath for the trace event data, it supports `${rotation}` and `${pid}`.

### `--trace-events-enabled`
<!-- YAML
added: v7.7.0
-->

Enables the collection of trace event tracing information.

### `--trace-exit`
<!-- YAML
added: v13.5.0
-->

Prints a stack trace whenever an environment is exited proactively, i.e. invoking `process.exit()`.

### `--trace-sigint`
<!-- YAML
added: v13.9.0
-->

Prints a stack trace on SIGINT.

### `--trace-sync-io`
<!-- YAML
added: v2.1.0
-->

Печатает трассировку стека каждый раз, когда обнаруживается синхронизированный ввод/вывод после первого запуска цикла событий.

### `--trace-tls`
<!-- YAML
added: v12.2.0
-->

Prints TLS packet trace information to `stderr`. This can be used to debug TLS connection problems.

### `--trace-uncaught`
<!-- YAML
added: v13.1.0
-->

Print stack traces for uncaught exceptions; usually, the stack trace associated with the creation of an `Error` is printed, whereas this makes Node.js also print the stack trace associated with throwing the value (which does not need to be an `Error` instance).

Enabling this option may affect garbage collection behavior negatively.

### `--trace-warnings`
<!-- YAML
added: v6.0.0
-->

Печать трассировок стека для процессных уведомлений (включая устаревшие версии).

### `--track-heap-objects`
<!-- YAML
added: v2.4.0
-->

Отслеживание распределения групп объектов для групповых снепшотов.

### `--unhandled-rejections=mode`
<!-- YAML
added: v12.0.0
-->

By default all unhandled rejections trigger a warning plus a deprecation warning for the very first unhandled rejection in case no [`unhandledRejection`][] hook is used.

Using this flag allows to change what should happen when an unhandled rejection occurs. One of three modes can be chosen:

* `strict`: Raise the unhandled rejection as an uncaught exception.
* `warn`: Always trigger a warning, no matter if the [`unhandledRejection`][] hook is set or not but do not print the deprecation warning.
* `none`: Silence all warnings.

### `--use-bundled-ca`, `--use-openssl-ca`
<!-- YAML
added: v6.11.0
-->

Use bundled Mozilla CA store as supplied by current Node.js version or use OpenSSL's default CA store. The default store is selectable at build-time.

Объединенное хранилище CA, предоставленное Node.js, является снепшотом хранилища Mozilla CA, исправленным во время запуска. Он идентичен на всех поддерживаемых платформах.

Использование хранилища OpenSSL допускает внешние модификации хранилища. Для большинства дистрибутивов Linux и BSD это хранилище поддерживается сопровождающими и системными администраторами. Расположение хранилища CA OpenSSL зависит от конфигурации библиотеки OpenSSL, но это может быть изменено с помощью переменных среды во время выполнения.

Смотрите `SSL_CERT_DIR` и `SSL_CERT_FILE`.

### `--use-largepages=mode`
<!-- YAML
added: v13.6.0
-->

Re-map the Node.js static code to large memory pages at startup. If supported on the target system, this will cause the Node.js static code to be moved onto 2 MiB pages instead of 4 KiB pages.

The following values are valid for `mode`:
* `off`: No mapping will be attempted. This is the default.
* `on`: If supported by the OS, mapping will be attempted. Failure to map will be ignored and a message will be printed to standard error.
* `silent`: If supported by the OS, mapping will be attempted. Failure to map will be ignored and will not be reported.

### `--v8-options`
<!-- YAML
added: v0.1.3
-->

Print V8 command line options.

### `--v8-pool-size=num`
<!-- YAML
added: v5.10.0
-->

Set V8's thread pool size which will be used to allocate background jobs.

If set to `0` then V8 will choose an appropriate size of the thread pool based on the number of online processors.

If the value provided is larger than V8's maximum, then the largest value will be chosen.

### `--zero-fill-buffers`
<!-- YAML
added: v6.0.0
-->

Automatically zero-fills all newly allocated [`Buffer`][] and [`SlowBuffer`][] instances.

### `-c`, `--check`
<!-- YAML
added:
  - v5.0.0
  - v4.2.0
changes:
  - version: v10.0.0
    pr-url: https://github.com/nodejs/node/pull/19600
    description: The `--require` option is now supported when checking a file.
-->

Проверка синтаксиса сценария без выполнения.

### `-e`, `--eval "script"`
<!-- YAML
added: v0.5.2
changes:
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Built-in libraries are now available as predefined variables.
-->

Оцените следующий аргумент как JavaScript. Модули, которые предопределены в REPL, также могут использоваться в `script`.

On Windows, using `cmd.exe` a single quote will not work correctly because it only recognizes double `"` for quoting. In Powershell or Git bash, both `'` and `"` are usable.

### `-h`, `--help`
<!-- YAML
added: v0.1.3
-->

Печать опций командной строки узла. Вывод этой опции менее подробен, чем этот документ.

### `-i`, `--interactive`
<!-- YAML
added: v0.7.7
-->

Открывает REPL, даже если stdin не является терминалом.

### `-p`, `--print "script"`
<!-- YAML
added: v0.6.4
changes:
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Built-in libraries are now available as predefined variables.
-->

Идентично `-e`, но печатает результат.

### `-r`, `--require module`
<!-- YAML
added: v1.6.0
-->

Предварительная загрузка указанного модуля при запуске.

Следует правилам разрешения модуля `require()`. `module` может быть путем к файлу или именем модуля узла.

### `-v`, `--version`
<!-- YAML
added: v0.1.3
-->

Печать версии узла.

## Переменные среды

### `NODE_DEBUG=module[,…]`
<!-- YAML
added: v0.1.32
-->

`','` - разделенный список основных модулей, которые должны печатать отладочную информацию.

### `NODE_DEBUG_NATIVE=module[,…]`

`','`-separated list of core C++ modules that should print debug information.

### `NODE_DISABLE_COLORS=1`
<!-- YAML
added: v0.3.0
-->

When set, colors will not be used in the REPL.

### `NODE_EXTRA_CA_CERTS=file`
<!-- YAML
added: v7.3.0
-->

Когда установлено, хорошо известные "корневые" CA (такие как VeriSign) будут расширены с дополнительными сертификатами в `file`. Файл должен состоять из одного или нескольких доверенных сертификатов в формате PEM. A message will be emitted (once) with [`process.emitWarning()`](process.html#process_process_emitwarning_warning_type_code_ctor) if the file is missing or malformed, but any errors are otherwise ignored.

Neither the well known nor extra certificates are used when the `ca` options property is explicitly specified for a TLS or HTTPS client or server.

This environment variable is ignored when `node` runs as setuid root or has Linux file capabilities set.

### `NODE_ICU_DATA=file`
<!-- YAML
added: v0.11.15
-->

Data path for ICU (`Intl` object) data. Расширит связанные данные при компиляции с поддержкой малого icu.

### `NODE_NO_WARNINGS=1`
<!-- YAML
added: v6.11.0
-->

Если установлено значение `1`, предупреждения процесса отключаются.

### `NODE_OPTIONS=options...`
<!-- YAML
added: v8.0.0
-->

A space-separated list of command line options. `options...` are interpreted before command line options, so command line options will override or compound after anything in `options...`. Node.js will exit with an error if an option that is not allowed in the environment is used, such as `-p` or a script file.

In case an option value happens to contain a space (for example a path listed in `--require`), it must be escaped using double quotes. Например:

```bash
NODE_OPTIONS='--require "./my path/file.js"'
```

A singleton flag passed as a command line option will override the same flag passed into `NODE_OPTIONS`:

```bash
# The inspector will be available on port 5555
NODE_OPTIONS='--inspect=localhost:4444' node --inspect=localhost:5555
```

A flag that can be passed multiple times will be treated as if its `NODE_OPTIONS` instances were passed first, and then its command line instances afterwards:

```bash
NODE_OPTIONS='--require "./a.js"' node --require "./b.js"
# is equivalent to:
node --require "./a.js" --require "./b.js"
```

Node.js options that are allowed are:
<!-- node-options-node start -->
* `--enable-fips`
* `--enable-source-maps`
* `--experimental-import-meta-resolve`
* `--experimental-json-modules`
* `--experimental-loader`
* `--experimental-modules`
* `--experimental-policy`
* `--experimental-repl-await`
* `--experimental-report`
* `--experimental-specifier-resolution`
* `--experimental-vm-modules`
* `--experimental-wasi-unstable-preview1`
* `--experimental-wasm-modules`
* `--force-context-aware`
* `--force-fips`
* `--frozen-intrinsics`
* `--heapsnapshot-signal`
* `--http-parser`
* `--icu-data-dir`
* `--input-type`
* `--insecure-http-parser`
* `--inspect-brk`
* `--inspect-port`, `--debug-port`
* `--inspect-publish-uid`
* `--inspect`
* `--max-http-header-size`
* `--napi-modules`
* `--no-deprecation`
* `--no-force-async-hooks-checks`
* `--no-warnings`
* `--openssl-config`
* `--pending-deprecation`
* `--policy-integrity`
* `--preserve-symlinks-main`
* `--preserve-symlinks`
* `--prof-process`
* `--redirect-warnings`
* `--report-directory`
* `--report-filename`
* `--report-on-fatalerror`
* `--report-on-signal`
* `--report-signal`
* `--report-uncaught-exception`
* `--require`, `-r`
* `--throw-deprecation`
* `--title`
* `--tls-cipher-list`
* `--tls-keylog`
* `--tls-max-v1.2`
* `--tls-max-v1.3`
* `--tls-min-v1.0`
* `--tls-min-v1.1`
* `--tls-min-v1.2`
* `--tls-min-v1.3`
* `--trace-deprecation`
* `--trace-event-categories`
* `--trace-event-file-pattern`
* `--trace-events-enabled`
* `--trace-exit`
* `--trace-sigint`
* `--trace-sync-io`
* `--trace-tls`
* `--trace-uncaught`
* `--trace-warnings`
* `--track-heap-objects`
* `--unhandled-rejections`
* `--use-bundled-ca`
* `--use-largepages`
* `--use-openssl-ca`
* `--v8-pool-size`
* `--zero-fill-buffers`
<!-- node-options-node end -->

V8 options that are allowed are:
<!-- node-options-v8 start -->
* `--abort-on-uncaught-exception`
* `--disallow-code-generation-from-strings`
* `--interpreted-frames-native-stack`
* `--max-old-space-size`
* `--perf-basic-prof-only-functions`
* `--perf-basic-prof`
* `--perf-prof-unwinding-info`
* `--perf-prof`
* `--stack-trace-limit`
<!-- node-options-v8 end -->

### `NODE_PATH=path[:…]`
<!-- YAML
added: v0.1.32
-->

`':'`- разделенный список директорий, предшествующих пути поиска модуля.

On Windows, this is a `';'`-separated list instead.

### `NODE_PENDING_DEPRECATION=1`
<!-- YAML
added: v8.0.0
-->

When set to `1`, emit pending deprecation warnings.

Pending deprecations are generally identical to a runtime deprecation with the notable exception that they are turned *off* by default and will not be emitted unless either the `--pending-deprecation` command line flag, or the `NODE_PENDING_DEPRECATION=1` environment variable, is set. Pending deprecations are used to provide a kind of selective "early warning" mechanism that developers may leverage to detect deprecated API usage.

### `NODE_PENDING_PIPE_INSTANCES=instances`

Set the number of pending pipe instance handles when the pipe server is waiting for connections. This setting applies to Windows only.

### `NODE_PRESERVE_SYMLINKS=1`
<!-- YAML
added: v7.1.0
-->

When set to `1`, instructs the module loader to preserve symbolic links when resolving and caching modules.

### `NODE_REDIRECT_WARNINGS=file`
<!-- YAML
added: v8.0.0
-->

Если это установлено, процессные уведомления будут отправляться данному файлу, а не печататься в stderr. Файл будет создаваться, если он не существует, и будет добавляться, если он существует. Если при попытке записать предупреждение в файл произойдет ошибка, предупреждение вместо этого будет записано в stderr. Это эквивалентно использованию флага командной строки `--redirect-warnings=file`.

### `NODE_REPL_HISTORY=file`
<!-- YAML
added: v3.0.0
-->

Путь к файлу, который используется для хранения постоянной истории REPL. Путь по умолчанию - `~/.node_repl_history`, который переопределяется этой переменной. Установка значения в пустую строку (`''` или `' '`) отключает постоянную историю REPL.

### `NODE_REPL_EXTERNAL_MODULE=file`
<!-- YAML
added: v13.0.0
-->

Path to a Node.js module which will be loaded in place of the built-in REPL. Overriding this value to an empty string (`''`) will use the built-in REPL.

### `NODE_TLS_REJECT_UNAUTHORIZED=value`

If `value` equals `'0'`, certificate validation is disabled for TLS connections. This makes TLS, and HTTPS by extension, insecure. The use of this environment variable is strongly discouraged.

### `NODE_V8_COVERAGE=dir`

When set, Node.js will begin outputting [V8 JavaScript code coverage](https://v8project.blogspot.com/2017/12/javascript-code-coverage.html) and [Source Map](https://sourcemaps.info/spec.html) data to the directory provided as an argument (coverage information is written as JSON to files with a `coverage` prefix).

`NODE_V8_COVERAGE` will automatically propagate to subprocesses, making it easier to instrument applications that call the `child_process.spawn()` family of functions. `NODE_V8_COVERAGE` can be set to an empty string, to prevent propagation.

#### Coverage Output

Coverage is output as an array of [ScriptCoverage](https://chromedevtools.github.io/devtools-protocol/tot/Profiler#type-ScriptCoverage) objects on the top-level key `result`:

```json
{
  "result": [
    {
      "scriptId": "67",
      "url": "internal/tty.js",
      "functions": []
    }
  ]
}
```

#### Source Map Cache

> Стабильность: 1 - экспериментальный

If found, Source Map data is appended to the top-level key `source-map-cache` on the JSON coverage object.

`source-map-cache` is an object with keys representing the files source maps were extracted from, and values which include the raw source-map URL (in the key `url`), the parsed Source Map V3 information (in the key `data`), and the line lengths of the source file (in the key `lineLengths`).

```json
{
  "result": [
    {
      "scriptId": "68",
      "url": "file:///absolute/path/to/source.js",
      "functions": []
    }
  ],
  "source-map-cache": {
    "file:///absolute/path/to/source.js": {
      "url": "./path-to-map.json",
      "data": {
        "version": 3,
        "sources": [
          "file:///absolute/path/to/original.js"
        ],
        "names": [
          "Foo",
          "console",
          "info"
        ],
        "mappings": "MAAMA,IACJC,YAAaC",
        "sourceRoot": "./"
      },
      "lineLengths": [
        13,
        62,
        38,
        27
      ]
    }
  }
}
```

### `OPENSSL_CONF=file`
<!-- YAML
added: v6.11.0
-->

Загрузка файла конфигурации OpenSSL при запуске. Среди прочего это может использоваться для включения FIPS-совместимого шифрования, если Node.js создан с `./configure
--openssl-fips`.

Если используется опция командной строки [`--openssl-config`][], переменная среда игнорируется.

### `SSL_CERT_DIR=dir`
<!-- YAML
added: v7.7.0
-->

Если включен `--use-openssl-ca`, это переопределяет и устанавливает каталог OpenSSL, содержащий доверенные сертификаты.

Be aware that unless the child environment is explicitly set, this environment variable will be inherited by any child processes, and if they use OpenSSL, it may cause them to trust the same CAs as node.

### `SSL_CERT_FILE=file`
<!-- YAML
added: v7.7.0
-->

Если включен `--use-openssl-ca`, это переопределяет и устанавливает файл OpenSSL, содержащий доверительные сертификаты.

Be aware that unless the child environment is explicitly set, this environment variable will be inherited by any child processes, and if they use OpenSSL, it may cause them to trust the same CAs as node.

### `UV_THREADPOOL_SIZE=size`

Set the number of threads used in libuv's threadpool to `size` threads.

Asynchronous system APIs are used by Node.js whenever possible, but where they do not exist, libuv's threadpool is used to create asynchronous node APIs based on synchronous system APIs. Node.js APIs that use the threadpool are:

* all `fs` APIs, other than the file watcher APIs and those that are explicitly synchronous
* asynchronous crypto APIs such as `crypto.pbkdf2()`, `crypto.scrypt()`, `crypto.randomBytes()`, `crypto.randomFill()`, `crypto.generateKeyPair()`
* `dns.lookup()`
* all `zlib` APIs, other than those that are explicitly synchronous

Because libuv's threadpool has a fixed size, it means that if for whatever reason any of these APIs takes a long time, other (seemingly unrelated) APIs that run in libuv's threadpool will experience degraded performance. In order to mitigate this issue, one potential solution is to increase the size of libuv's threadpool by setting the `'UV_THREADPOOL_SIZE'` environment variable to a value greater than `4` (its current default value). For more information, see the [libuv threadpool documentation](http://docs.libuv.org/en/latest/threadpool.html).