# Ruta de Acceso

<!--introduced_in=v0.10.0-->

> Estability: 2 - Estable

El módulo `path` proporciona utilidades para trabajar con rutas de archivos y directorios. Se puede acceder a él utilizando:

```js
const path = require('path');
```

## Windows vs. POSIX

La operación predeterminada del módulo `path` varía según el sistema operativo en el que una aplicación de Node.js está ejecutándose. Específicamente, cuando se ejecuta en el sistema operativo Windows, el módulo `path` va a asumir que las rutas estilo Windows están siendo usadas.

So using `path.basename()` might yield different results on POSIX and Windows:

En POSIX:

```js
path.basename('C:\\temp\\myfile.html');
// Devuelve: 'C:\\temp\\myfile.html'
```

En Windows:

```js
path.basename('C:\\temp\\myfile.html');
// Devuelve: 'myfile.html'
```

Para alcanzar resultados consistentes cuando se trabaja con rutas de archivo Windows en cualquier sistema operativo, utilice [`path.win32`][]:

En POSIX y Windows:

```js
path.win32.basename('C:\\temp\\myfile.html');
// Devuelve: 'myfile.html'
```

Para alcanzar resultados consistentes cuando se trabaja con rutas de archivo POSIX en cualquier sistema operativos, utilice [`path.posix`][]:

En POSIX y Windows:

```js
path.posix.basename('/tmp/myfile.html');
// Devuelve: 'myfile.html'
```

*Nota:* En Windows, Node.js sigue el concepto de directorio de trabajo por disco. Este comportamiento puede ser observado cuando se usa una ruta de disco sin un backslash. For example `path.resolve('c:\\')` can potentially return a different result than `path.resolve('c:')`. Para más información, consulte [esta página de MSDN](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365247.aspx#fully_qualified_vs._relative_paths).

## path.basename(path[, ext])

<!-- YAML
added: v0.1.25
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

* `path` {string}
* `ext` {string} Una extensión de archivo opcional
* Devuelve: {string}

Los métodos `path.basename()` devuelven la última porción de un `path`, similar al comando Unix `basename`. Los separadores de directorios de seguimiento son ignorados, vea [`path.sep`][].

For example:

```js
path.basename('/foo/bar/baz/asdf/quux.html');
// Devuelve: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// Devuelve: 'quux'
```

Se produce un [`TypeError`][] si `path` no es una string o si `ext` es dado y no es una string.

## path.delimiter

<!-- YAML
added: v0.9.3
-->

* {string}

Proporciona el delimitador de ruta específico de la plataforma:

* `;` para Windows
* `:` para POSIX

Por ejemplo, en POSIX:

```js
console.log(process.env.PATH);
// Devuelve: '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

process.env.PATH.split(path.delimiter);
// Devuelve: ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
```

En Windows:

```js
console.log(process.env.PATH);
// Imprime: 'C:\Windows\system32;C:\Windows;C:\Program Files\node\'

process.env.PATH.split(path.delimiter);
// Devuelve ['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\node\\']
```

## path.dirname(path)

<!-- YAML
added: v0.1.16
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

* `path` {string}
* Devuelve: {string}

El método `path.dirname()` devuelve el nombre del directorio de un `path`, similar al comando de Unix `dirname`. Los separadores de directorios de seguimiento son ignorados, vea [`path.sep`][].

For example:

```js
path.dirname('/foo/bar/baz/asdf/quux');
// Devuelve: '/foo/bar/baz/asdf'
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.extname(path)

<!-- YAML
added: v0.1.25
changes:

  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/5348
    description: Passing a non-string as the `path` argument will throw now.
-->

* `path` {string}
* Devuelve: {string}

El método `path.extname()` devuelve la extensión de `path`, desde la última ocurrencia del carácter `.` (punto) hasta el final de la string en la última porción de `path`. Si no hay ningún `.` en la última porción de `path`, o si el primer carácter del nombre base de `path` (vea `path.basename()`) es `.`, entonces una string vacía es devuelta.

For example:

```js
path.extname('index.html');
// Devuelve: '.html'

path.extname('index.coffee.md');
// Devuelve: '.md'

path.extname('index.');
// Devuelve: '.'

path.extname('index');
// Devuelve: ''

path.extname('.index');
// Devuelve: ''
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.format(pathObject)

<!-- YAML
added: v0.11.15
-->

* `pathObject` {Object} 
  * `dir` {string}
  * `root` {string}
  * `base` {string}
  * `name` {string}
  * `ext` {string}
* Devuelve: {string}

El método `path.format()` retorna un string de ruta de un objeto. Esto es lo opuesto de [`path.parse()`][].

Al proporcionar propiedades al `pathObject`, recuerde que hay combinaciones donde una propiedad tiene prioridad sobre otra:

* `pathObject.root` es ignorado si `pathObject.dir` es proporcionado
* `pathObject.ext` y `pathObject.name` son ignorados si `pathObject.base` existe

Por ejemplo, en POSIX:

```js
// Si `dir`, `root` y `base` son propocionados,
// `${dir}${path.sep}${base}`
// será devuelto. `root` es ignorado.
path.format({
  root: '/ignored',
  dir: '/home/user/dir',
  base: 'file.txt'
});
// Devuelve: '/home/user/dir/file.txt'

// `root` será usado si `dir` no es especificado.
// Si solo `root` es proporcionado o`dir` es igual a `root` entonces el
// separador de plataforma no va a ser incluido. `ext` será ignorado.
path.format({
  root: '/',
  base: 'file.txt',
  ext: 'ignored'
});
// Devuelve: '/file.txt'

// `name` + `ext` será usado si `base` no es especificado.
path.format({
  root: '/',
  name: 'file',
  ext: '.txt'
});
// Devuelve: '/file.txt'
```

En Windows:

```js
path.format({
  dir: 'C:\\path\\dir',
  base: 'file.txt'
});
// Devuelve: 'C:\\path\\dir\\file.txt'
```

## path.isAbsolute(path)

<!-- YAML
added: v0.11.2
-->

* `path` {string}
* Retorna: {boolean}

El método `path.isAbsolute()` determina si `path` es una ruta absoluta.

Si el `path` dado es un string sin extensión, `false` va a ser retornado.

Por ejemplo, en POSIX:

```js
path.isAbsolute('/foo/bar'); // true
path.isAbsolute('/baz/..');  // true
path.isAbsolute('qux/');     // false
path.isAbsolute('.');        // false
```

En Windows:

```js
path.isAbsolute('//server');    // true
path.isAbsolute('\\\\server');  // true
path.isAbsolute('C:/foo/..');   // true
path.isAbsolute('C:\\foo\\..'); // true
path.isAbsolute('bar\\baz');    // false
path.isAbsolute('bar/baz');     // false
path.isAbsolute('.');           // false
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.join([...paths])

<!-- YAML
added: v0.1.16
-->

* `...paths` {string} Una secuencia de segmentos de ruta
* Devuelve: {string}

El método `path.join()` une a todos los segmentos `path` dados usando el separador específico de plataforma como delimitador, luego normaliza la ruta resultante.

Los segmentos `path` sin extensión son ignorados. Si la string de ruta unida es una string de longitud cero, entonces `'.'` será devuelto, representando al directorio de trabajo actual.

For example:

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// Devuelve: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar');
// arroja 'TypeError: Path debe ser una string. Recibido {}'
```

Un [`TypeError`][] va a ser producido si cualquiera de los segmentos de ruta no es un string.

## path.normalize(path)

<!-- YAML
added: v0.1.23
-->

* `path` {string}
* Devuelve: {string}

El método `path.normalize()` normaliza el `path` dado, resolviendo los segmentos `'..'` y `'.'`.

Cuando múltiples carácteres de separación de segmentos de ruta secuenciales son encontrados (p. ej. `/` ya sea en POSIX `` o `/` en Windows), son reemplazados por una sola instancia del separador de segmento de ruta específico de la plataforma (`/` en POSIX y `` en Windows). Los separadores de seguimiento son preservados.

Si el `path` es una string de longitud cero, `'.'` es devuelto, representando el directorio de trabajo actual.

Por ejemplo, en POSIX:

```js
path.normalize('/foo/bar//baz/asdf/quux/..');
// Devuelve: '/foo/bar/baz/asdf'
```

En Windows:

```js
path.normalize('C:\\temp\\\\foo\\bar\\..\\');
// Devuelve: 'C:\\temp\\foo\\'
```

Ya que Windows reconoce múltiples separadores de ruta, ambos separadores van a ser reemplazados por instancias del separador preferido de Windows (``):

```js
path.win32.normalize('C:////temp\\\\/\\/\\/foo/bar');
// Devuelve: 'C:\\temp\\foo\\bar'
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.parse(path)

<!-- YAML
added: v0.11.15
-->

* `path` {string}
* Devuelve: {Object}

El método `path.parse()` devuelve un objeto cuyas propiedades representan elementos significativos del `path`. Los separadores de directorios de seguimiento son ignorados, vea [`path.sep`][].

El objeto retornado va a tener las siguientes propiedades:

* `dir` {string}
* `root` {string}
* `base` {string}
* `name` {string}
* `ext` {string}

Por ejemplo, en POSIX:

```js
path.parse('/home/user/dir/file.txt');
// Retorna:
// { root: '/',
//   dir: '/home/user/dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file' }
```

```text
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
"  /    home/user/dir / file  .txt "
└──────┴──────────────┴──────┴─────┘
(todos los espacios en la línea "" deben ser ignorados — ellos están solamente para el formato)
```

En Windows:

```js
path.parse('C:\\path\\dir\\file.txt');
// Retorna:
// { root: 'C:\\',
//   dir: 'C:\\path\\dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file' }
```

```text
┌─────────────────────┬────────────┐
│          dir        │    base    │
├──────┬              ├──────┬─────┤
│ root │              │ name │ ext │
" C:\      path\dir   \ file  .txt "
└──────┴──────────────┴──────┴─────┘
(todos los espacios en la línea "" deben ser ignorados — ellos están solamente para el formato)
```

Un [`TypeError`][] es producido si `path` no es un string.

## path.posix

<!-- YAML
added: v0.11.15
-->

* {Object}

La propiedad `path.posix` proporciona acceso a implementaciones de POSIX específicas de los métodos `path`.

## path.relative(from, to)

<!-- YAML
added: v0.5.0
changes:

  - version: v6.8.0
    pr-url: https://github.com/nodejs/node/pull/8523
    description: On Windows, the leading slashes for UNC paths are now included
                 in the return value.
-->

* `from` {string}
* `to` {string}
* Devuelve: {string}

El método `path.relative()` devuelve la ruta relativa de `from` a `to` según el directorio de trabajo actual. Si tanto `from` como `to` se resuelven para la misma ruta (después de llamar a `path.resolve()` en cada uno), un string de longitud cero es devuelto.

Si una string de longitud cero es pasado como `from` o `to`, el directorio de trabajo actual será usado en lugar de las strings de longitud cero.

Por ejemplo, en POSIX:

```js
path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb');
// Devuelve: '../../impl/bbb'
```

En Windows:

```js
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb');
// Devuelve: '..\\..\\impl\\bbb'
```

Un [`TypeError`][] es arrojado si `from` o `to` no son strings.

## path.resolve([...paths])

<!-- YAML
added: v0.3.4
-->

* `...paths` {string} Una secuencia de rutas o segmentos de ruta
* Devuelve: {string}

El método `path.resolve()` resuelve una secuencia de rutas o segmentos de rutas en una ruta absoluta.

La secuencia dada de rutas es procesada de derecha a izquierda, con cada subsecuente `path` antepuesto hasta que una ruta absoluta sea construida. Por ejemplo, dada la secuencia de segmentos de ruta: `/foo`, `/bar`, `baz`, llamar a `path.resolve('/foo', '/bar', 'baz')`, devolvería `/bar/baz`.

Si después de procesar todos los segmentos `path` aún no se ha generado una ruta absoluta, el directorio de trabajo actual es usado.

La ruta resultante es normalizada y los slashes de seguimiento son removidos, a menos que la ruta sea resuelta en el directorio root.

Los segmentos `path` sin extensión son ignorados.

Si ningún segmento `path` es pasado, `path.resolve()` devolverá la ruta absoluta del directorio de trabajo actual.

For example:

```js
path.resolve('/foo/bar', './baz');
// Devuelve: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/');
// Devuelve: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
// si el directorio de trabajo actual es /home/myself/node,
// esto devuelve '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

Un [`TypeError`][] es producido si cualquiera de los argumentos no es un string.

## path.sep

<!-- YAML
added: v0.7.9
-->

* {string}

Provee el separador de segmento de ruta específico de la plataforma:

* `` en Windows
* `/` en POSIX

Por ejemplo, en POSIX:

```js
'foo/bar/baz'.split(path.sep);
// Devuelve: ['foo', 'bar', 'baz']
```

En Windows:

```js
'foo\\bar\\baz'.split(path.sep);
// Devuelve: ['foo', 'bar', 'baz']
```

*Nota*: En Windows, tanto la barra inclinada (`/`) como la barra inversa (``) son aceptadas como separadores de segmentos de ruta; sin embargo, los métodos `path` solo agregan barras inversas (``).

## path.win32

<!-- YAML
added: v0.11.15
-->

* {Object}

La propiedad `path.win32` proporciona acceso a implementaciones específicas para Windows de los métodos `path`.