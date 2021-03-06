# Buffer

<!--introduced_in=v0.10.0-->

> Estabilidade: 2 - estável

Prior to the introduction of [`TypedArray`], the JavaScript language had no mechanism for reading or manipulating streams of binary data. The `Buffer` class was introduced as part of the Node.js API to enable interaction with octet streams in TCP streams, file system operations, and other contexts.

With [`TypedArray`] now available, the `Buffer` class implements the [`Uint8Array`] API in a manner that is more optimized and suitable for Node.js.

Instâncias da classe `Buffer` são semelhantes a arrays de inteiros, mas correspondem a tamanho fixo, alocação de memória bruta fora do V8 heap. The size of the `Buffer` is established when it is created and cannot be changed.

The `Buffer` class is within the global scope, making it unlikely that one would need to ever use `require('buffer').Buffer`.

Examples:

```js
// Cria um Buffer de comprimento 10 preenchido de zeros.
const buf1 = Buffer.alloc(10);

// Cria um Buffer de comprimento 10, cheio de 0x1.
const buf2 = Buffer.alloc(10, 1);

// Cria um buffer não inicializado de comprimento 10.
// Isto é mais rápido que chamar Buffer.alloc(), mas a instância retornada do
// Buffer pode conter dados antigos que precisam ser
// sobrescrita usando fill() ou write().
const buf3 = Buffer.allocUnsafe(10);

// Cria um Buffer contendo [0x1, 0x2, 0x3].
const buf4 = Buffer.from([1, 2, 3]);

// Cria um Buffer contendo UTF-8 bytes [0x74, 0xc3, 0xa9, 0x73, 0x74].
const buf5 = Buffer.from('tést');

// Cria um Buffer contendo Latin-1 bytes [0x74, 0xe9, 0x73, 0x74].
const buf6 = Buffer.from('tést', 'latin1');
```

## `Buffer.from()`, `Buffer.alloc()`, e `Buffer.allocUnsafe()`

In versions of Node.js prior to v6, `Buffer` instances were created using the `Buffer` constructor function, which allocates the returned `Buffer` differently based on what arguments are provided:

* Passing a number as the first argument to `Buffer()` (e.g. `new Buffer(10)`), allocates a new `Buffer` object of the specified size. Prior to Node.js 8.0.0, the memory allocated for such `Buffer` instances is *not* initialized and *can contain sensitive data*. Such `Buffer` instances *must* be subsequently initialized by using either [`buf.fill(0)`][`buf.fill()`] or by writing to the `Buffer` completely. While this behavior is *intentional* to improve performance, development experience has demonstrated that a more explicit distinction is required between creating a fast-but-uninitialized `Buffer` versus creating a slower-but-safer `Buffer`. Starting in Node.js 8.0.0, `Buffer(num)` and `new Buffer(num)` will return a `Buffer` with initialized memory.
* Passando uma string, array ou `Buffer` como o primeiro argumento copia os dados do objeto passado no `Buffer`.
* Passando um [`ArrayBuffer`] ou um [`SharedArrayBuffer`] retorna um `Buffer` que compartilha a memória atribuída com o array buffer informado.

Because the behavior of `new Buffer()` changes significantly based on the type of value passed as the first argument, applications that do not properly validate the input arguments passed to `new Buffer()`, or that fail to appropriately initialize newly allocated `Buffer` content, can inadvertently introduce security and reliability issues into their code.

To make the creation of `Buffer` instances more reliable and less error prone, the various forms of the `new Buffer()` constructor have been **deprecated** and replaced by separate `Buffer.from()`, [`Buffer.alloc()`], and [`Buffer.allocUnsafe()`] methods.

*Os desenvolvedores devem migrar todas as utilizações existentes dos construtores `new Buffer()` para uma dessas novas APIs.*

* [`Buffer.from(array)`] returns a new `Buffer` containing a *copy* of the provided octets.
* [`Buffer.from(arrayBuffer[, byteOffset [, length]])`][`Buffer.from(arrayBuffer)`] returns a new `Buffer` that *shares* the same allocated memory as the given [`ArrayBuffer`].
* [`Buffer.from(buffer)`] returns a new `Buffer` containing a *copy* of the contents of the given `Buffer`.
* [`Buffer.from(string[, encoding])`][`Buffer.from(string)`] returns a new `Buffer` containing a *copy* of the provided string.
* [`Buffer.alloc(size[, fill[, encoding]])`][`Buffer.alloc()`] returns a "filled" `Buffer` instance of the specified size. This method can be significantly slower than [`Buffer.allocUnsafe(size)`][`Buffer.allocUnsafe()`] but ensures that newly created `Buffer` instances never contain old and potentially sensitive data.
* [`Buffer.allocUnsafe(size)`][`Buffer.allocUnsafe()`] and [`Buffer.allocUnsafeSlow(size)`][`Buffer.allocUnsafeSlow()`] each return a new `Buffer` of the specified `size` whose content *must* be initialized using either [`buf.fill(0)`][`buf.fill()`] or written to completely.

As instâncias `Buffer` retornadas por [`Buffer.allocUnsafe()`] *podem* ser alocadas fora de um conjunto de memórias internas compartilhadas se `size` for menor ou igual a metade de [`Buffer.poolSize`]. Instâncias retornadas por [`Buffer.allocUnsafeSlow()`] *nunca* usam o conjunto memória interna compartilhado.

### A opção `--zero-fill-buffers` linha de comando

<!-- YAML
added: v5.10.0
-->

Node.js can be started using the `--zero-fill-buffers` command line option to force all newly allocated `Buffer` instances created using either `new Buffer(size)`, [`Buffer.allocUnsafe()`], [`Buffer.allocUnsafeSlow()`] or `new SlowBuffer(size)` to be *automatically zero-filled* upon creation. Use of this flag *changes the default behavior* of these methods and *can have a significant impact* on performance. Use of the `--zero-fill-buffers` option is recommended only when necessary to enforce that newly allocated `Buffer` instances cannot contain potentially sensitive data.

Exemplo:

```txt
$ node --zero-fill-buffers
> Buffer.allocUnsafe(5);
<Buffer 00 00 00 00 00>
```

### O que faz `Buffer.allocUnsafe()` e `Buffer.allocUnsafew()` "Não seguro"?

Ao chamar [`Buffer.allocUnsafe()`] e [`Buffer.allocUnsafeSlow()`], o segmento de memória alocada *não é inicializado* (não é zerado). Enquanto este design torna a alocação de memória muito rápida, o segmento alocado de memória pode conter dados antigos que são potencialmente sensíveis. Usando um `Buffer` criado por [`Buffer.allocUnsafe()`] sem substituir *completamente* a memória pode permitir que estes dados antigos sejam vazados quando a memória `Buffer` é lida.

Embora existam vantagens de desempenho claras para usar [`Buffer.allocUnsafe()`], cuidado extra *deve* ser usado para evitar introduzir vulnerabilidades de segurança na aplicação.

## Codificações de Buffers e Caracteres

<!-- YAML
changes:

  - version: v6.4.0
    pr-url: https://github.com/nodejs/node/pull/7111
    description: Introduced `latin1` as an alias for `binary`.
  - version: v5.0.0
    pr-url: https://github.com/nodejs/node/pull/2859
    description: Removed the deprecated `raw` and `raws` encodings.
-->

`Buffer` instances are commonly used to represent sequences of encoded characters such as UTF-8, UCS2, Base64, or even Hex-encoded data. It is possible to convert back and forth between `Buffer` instances and ordinary JavaScript strings by using an explicit character encoding.

Exemplo:

```js
const buf = Buffer.from('hello world', 'ascii');

// Prints: 68656c6c6f20776f726c64
console.log(buf.toString('hex'));

// Prints: aGVsbG8gd29ybGQ=
console.log(buf.toString('base64'));
```

As codificações de caracteres atualmente suportadas pelo Node.js incluem:

* `'ascii'` - Somente para dados ASCII de 7 bits. Esta codificação é rápida e tirará o bit alto se definido.

* `'utf8'` - Codificação de multibyte de caracteres Unicode. Muitas páginas da web e outros formatos de documentos usam UTF-8.

* `'utf16le'` - 2 ou 4 bytes, caracteres little-endian Unicode. Pares adicionais (U+1000para U+10FFFF) são suportados.

* `'ucs2'` - Alias de `'utf16le'`.

* `'base64'` - Codificação Base64. Ao criar um `Buffer` de uma string, esta codificação também aceitará corretamente "URL e Filename Safe Alphabet" como especificado em [RFC4648, Seção 5](https://tools.ietf.org/html/rfc4648#section-5).

* `'latin1'` - Uma maneira de codificar o `Buffer` em uma string codificada de um byte (conforme definido pela IANA em [RFC1345](https://tools.ietf.org/html/rfc1345), página 63, para ser o bloco de complemento latino-1 e códigos de controle C0/C1).

* `'binary'` - Alias para `'latin1'`.

* `'hex'` - Codificar cada byte como dois caracteres hexadecimal.

*Note*: Today's browsers follow the [WHATWG Encoding Standard](https://encoding.spec.whatwg.org/) which aliases both `'latin1'` and `'ISO-8859-1'` to `'win-1252'`. This means that while doing something like `http.get()`, if the returned charset is one of those listed in the WHATWG specification it is possible that the server actually returned `'win-1252'`-encoded data, and using `'latin1'` encoding may incorrectly decode the characters.

## Buffers e TypedArray

<!-- YAML
changes:

  - version: v3.0.0
    pr-url: https://github.com/nodejs/node/pull/2002
    description: The `Buffer`s class now inherits from `Uint8Array`.
-->

Instâncias `Buffer` também são instâncias [`Uint8Array`]. However, there are subtle incompatibilities with [`TypedArray`]. For example, while [`ArrayBuffer#slice()`] creates a copy of the slice, the implementation of [`Buffer#slice()`][`buf.slice()`] creates a view over the existing `Buffer` without copying, making [`Buffer#slice()`][`buf.slice()`] far more efficient.

É também possível criar novas instâncias [`TypedArray`] de um `Buffer` com as seguintes ressalvas:

1. A memória do objeto `Buffer` é copiada para a [`TypedArray`], não compartilhada.

2. A memória do objeto `Buffer` é interpretada como uma array de elementos distintos, e não como uma byte array do tipo alvo. Isto é, `new Uint32Array(Buffer.from([1, 2, 3, 4]))` cria um 4-elemento [`Uint32Array`] com elementos `[1, 2, 3, 4]`, não um [`Uint32Array`] com um único elemento `[0x1020304]` ou `[0x4030201]`.

It is possible to create a new `Buffer` that shares the same allocated memory as a [`TypedArray`] instance by using the TypeArray object's `.buffer` property.

Exemplo:

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Copies the contents of `arr`
const buf1 = Buffer.from(arr);

// Shares memory with `arr`
const buf2 = Buffer.from(arr.buffer);

// Prints: <Buffer 88 a0>
console.log(buf1);

// Prints: <Buffer 88 13 a0 0f>
console.log(buf2);

arr[1] = 6000;

// Prints: <Buffer 88 a0>
console.log(buf1);

// Prints: <Buffer 88 13 70 17>
console.log(buf2);
```

Note que ao criar um `Buffer` usando um `.buffer` [`TypedArray`], é possível usar apenas uma porção dos parâmetros subjacentes [`ArrayBuffer`] passando pelos parâmetros `byteOffset` e `length`.

Exemplo:

```js
const arr = new Uint16Array(20);
const buf = Buffer.from(arr.buffer, 0, 16);

// Prints: 16
console.log(buf.length);
```

O `Buffer.from()` e [`TypedArray.from()`] tem assinaturas e implementações diferentes. Especificamente, as variantes [`TypedArray`] aceitam um segundo argumento que é uma função de mapeamento que é invocada em todos os elementos do array digitado:

* `TypedArray.de(source[, mapFn[, thisArg]])`

O método `Buffer.from()` no entanto, não suporta o uso de uma função de mapeamento:

* [`Buffer.from(array)`]
* [`Buffer.from(buffer)`]
* [`Buffer.from(arrayBuffer[, byteOffset [, length]])`][`Buffer.from(arrayBuffer)`]
* [`Buffer.from(string[, encoding])`][`Buffer.from(string)`]

## Buffers e iteração

Instâncias `Buffer` podem ser iteradas usando a syntax `for..of`:

Exemplo:

```js
const buf = Buffer.from([1, 2, 3]);

// Imprime:
//   1
//   2
//   3
for (const b of buf) {
  console.log(b);
}
```

Além disso, os métodos [`buf.values()`], [`buf.keys()`], e [`buf.entries()`] podem ser usados para criar iteradores.

## Classe: Buffer

A classe `Buffer` é um tipo global para lidar com dados binários diretamente. Pode ser construído de várias maneiras.

### new Buffer(array)

<!-- YAML
deprecated: v6.0.0
changes:

  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> Estabilidade: 0 - Descontinuada: Use [`Buffer.from(array)`] em vez disso.

* `array` {integer[]} Uma array de bytes para copiar.

Aloca um novo `Buffer` usando um `array` de octets.

Exemplo:

```js
// Cria um novo Buffer contendo UTF-8 bytes da string 'buffer'
const buf = new Buffer([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

### new Buffer(arrayBuffer[, byteOffset[, length]])

<!-- YAML
added: v3.0.0
deprecated: v6.0.0
changes:

  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
  - version: v6.0.0
    pr-url: https://github.com/nodejs/node/pull/4682
    description: The `byteOffset` and `length` parameters are supported now.
-->

> Stability: 0 - Deprecated: Use [`Buffer.from(arrayBuffer[, byteOffset [, length]])`][`Buffer.from(arrayBuffer)`] instead.

* `arrayBuffer` {ArrayBuffer|SharedArrayBuffer} Um [`ArrayBuffer`], [`SharedArrayBuffer`] ou a propriedade `.buffer` de um [`TypedArray`].
* `byteOffset` {integer} Index do primeiro byte para expor. **Default:** `0`.
* `length` {integer} Número de bytes para expor. **Default:** `arrayBuffer.length - byteOffset`.

Isso cria uma visão do [`ArrayBuffer`] ou [`SharedArrayBuffer`] sem copiar a memória subjacente. Por exemplo, quando passou uma referência para a propriedade `.buffer` de uma instância [`TypedArray`], o `Buffer` recém-criado irá compartilhar a mesma memória alocada que o [`TypedArray`].

Os argumentos opcionais `byteOffset` e `length` especificam um intervalo de memória dentro de `arrayBuffer` que será compartilhado pelo `Buffer`.

Exemplo:

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`
const buf = new Buffer(arr.buffer);

// Prints: <Buffer 88 13 a0 0f>
console.log(buf);

// Changing the original Uint16Array changes the Buffer also
arr[1] = 6000;

// Prints: <Buffer 88 13 70 17>
console.log(buf);
```

### new Buffer(buffer)

<!-- YAML
deprecated: v6.0.0
changes:

  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> Estabilidade: 0 - Descontinuada: Use [`Buffer.from(buffer)`] em vez disso.

* `buffer` {Buffer|Uint8Array} An existing `Buffer` or [`Uint8Array`] from which to copy data.

Copia os dados passados `buffer` em uma nova instância `Buffer`.

Exemplo:

```js
const buf1 = new Buffer('buffer');
const buf2 = new Buffer(buf1);

buf1[0] = 0x61;

// Prints: auffer
console.log(buf1.toString());

// Prints: buffer
console.log(buf2.toString());
```

### new Buffer(size)

<!-- YAML
deprecated: v6.0.0
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/12141
    description: new Buffer(size) will return zero-filled memory by default.
  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> Estabilidade: 0 - Descontinuada,: Use [`Buffer.alloc()`] em vez disso (também veja [`Buffer.allocUnsafe()`]).

* `size` {integer} O comprimento desejado do novo `Buffer`.

Aloca um novo `Buffer` de `size` bytes. If the `size` is larger than [`buffer.constants.MAX_LENGTH`] or smaller than 0, a [`RangeError`] will be thrown. A zero-length `Buffer` will be created if `size` is 0.

Prior to Node.js 8.0.0, the underlying memory for `Buffer` instances created in this way is *not initialized*. The contents of a newly created `Buffer` are unknown and *may contain sensitive data*. Use [`Buffer.alloc(size)`][`Buffer.alloc()`] instead to initialize a `Buffer` to zeroes.

Exemplo:

```js
const buf = new Buffer(10);

// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
console.log(buf);
```

### new Buffer(string[, encoding])

<!-- YAML
deprecated: v6.0.0
changes:

  - version: v7.2.1
    pr-url: https://github.com/nodejs/node/pull/9529
    description: Calling this constructor no longer emits a deprecation warning.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8169
    description: Calling this constructor emits a deprecation warning now.
-->

> Estabilidade: 0 - Descontinuada: Use [`Buffer.from(string[, encoding])`][`Buffer.from(string)`] em vez disso.

* `string` {string} String para codificar.
* `encoding` {string} A codificação da `string`. **Default:** `'utf8'`.

Cria uma nova `Buffer` que contém `string`. The `encoding` parameter identifies the character encoding of `string`.

Examples:

```js
const buf1 = new Buffer('this is a tést');

// Prints: this is a tést
console.log(buf1.toString());

// Prints: this is a tC)st
console.log(buf1.toString('ascii'));

const buf2 = new Buffer('7468697320697320612074c3a97374', 'hex');

// Prints: this is a tést
console.log(buf2.toString());
```

### Método de Classe: Buffer.alloc(size[, fill[, encoding]])

<!-- YAML
added: v5.10.0
changes:

  - version: v8.9.3
    pr-url: https://github.com/nodejs/node/pull/17428
    description: Specifying an invalid string for `fill` now results in a
                 zero-filled buffer.
-->

* `size` {integer} O comprimento desejado do novo `Buffer`.
* `fill` {string|Buffer|integer} Um valor para pré-preencher com o novo `Buffer`. **Default:** `0`.
* `encoding` {string} Se `fill` é uma string, esta é sua codificação. **Default:** `'utf8'`.

Aloca um novo `Buffer` de `size` bytes. Se `fill` for `undefined`, o `Buffer` será *zero-preenchido*.

Exemplo:

```js
const buf = Buffer.alloc(5);

// Prints: <Buffer 00 00 00 00 00>
console.log(buf);
```

Aloca um novo `Buffer` de `size` bytes. If the `size` is larger than [`buffer.constants.MAX_LENGTH`] or smaller than 0, a [`RangeError`] will be thrown. A zero-length `Buffer` will be created if `size` is 0.

Se `fill` for especificado, o alocado `Buffer` será inicializado chamando [`buf.fill(fill)`][`buf.fill()`].

Exemplo:

```js
const buf = Buffer.alloc(5, 'a');

// Prints: <Buffer 61 61 61 61 61>
console.log(buf);
```

Se ambos `fill` e `encoding` forem especificados, o `Buffer` alocado será inicializado chamando [`buf.fill(fill)`][`buf.fill()`].

Exemplo:

```js
const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

// Prints: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
console.log(buf);
```

Chamar [`Buffer.alloc()`] pode ser significativamente mais lento que a alternativa [`Buffer.allocUnsafe()`], mas garante que a instância do conteúdo do `Buffer` recém-criado *nunca conterá dados confidenciais*.

Um `TypeError` será lançado se `size` não é um número.

### Método de Classe: Buffer.allocUnsafe(size)

<!-- YAML
added: v5.10.0
changes:

  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/7079
    description: Passing a negative `size` will now throw an error.
-->

* `size` {integer} O comprimento desejado do novo `Buffer`.

Aloca um novo `Buffer` de `size` bytes. If the `size` is larger than [`buffer.constants.MAX_LENGTH`] or smaller than 0, a [`RangeError`] will be thrown. A zero-length `Buffer` will be created if `size` is 0.

A memória subjacente para instâncias `Buffer` criadas desta forma *não é inicializado*. O conteúdo de um `Buffer` recém criado é desconhecido e *pode conter dados sensíveis*. Use [`Buffer.alloc()`] instead to initialize `Buffer` instances to zeroes.

Exemplo:

```js
const buf = Buffer.allocUnsafe(10);

// Prints: (contents may vary): <Buffer a0 8b 28 3f 01 00 00 00 50 32>
console.log(buf);

buf.fill(0);

// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
console.log(buf);
```

Um `TypeError` será lançado se `size` não é um número.

Note que o módulo `Buffer` pré-aloca uma instância interna de `Buffer` de tamanho [`Buffer.poolSize`] que é usado como um pool para a alocação rápida de novas Instâncias de `Buffer` criadas usando [`Buffer.allocUnsafe()`] e as descontinuações `new Buffer(size)` construtor apenas quando `size` for menor ou igual a `Buffer.poolSize >> 1` (piso de [`Buffer.poolSize`] dividido por dois).

O uso deste pool de memória interna pré-alocada é uma diferença de chave entre chamar `Buffer.alloc(size, fill)` vs. `Buffer.allocUnsafe(size).fill(fill)`. Especificamente, `Buffer.alloc(size, fill)` *nunca* usará o `Buffer` pool interno, enquanto `Buffer.allocUnsafe(size).fill(fill)` *irá* usar o `Buffer` pool interno se `size` for menor ou igual a metade de [`Buffer.poolSize`]. A diferença é sutil, mas pode ser importante quando uma aplicação requer o desempenho adicional que [`Buffer.allocUnsafe()`] fornece.

### Método Classe: Buffer.allocUnsafeSlow(size)

<!-- YAML
added: v5.12.0
-->

* `size` {integer} O comprimento desejado do novo `Buffer`.

Aloca um novo `Buffer` de `size` bytes. If the `size` is larger than [`buffer.constants.MAX_LENGTH`] or smaller than 0, a [`RangeError`] will be thrown. A zero-length `Buffer` will be created if `size` is 0.

A memória subjacente para instâncias `Buffer` criadas desta forma *não é inicializado*. O conteúdo de um `Buffer` recém criado é desconhecido e *pode conter dados sensíveis*. Use [`buf.fill(0)`][`buf.fill()`] to initialize such `Buffer` instances to zeroes.

When using [`Buffer.allocUnsafe()`] to allocate new `Buffer` instances, allocations under 4KB are, by default, sliced from a single pre-allocated `Buffer`. This allows applications to avoid the garbage collection overhead of creating many individually allocated `Buffer` instances. This approach improves both performance and memory usage by eliminating the need to track and cleanup as many `Persistent` objects.

However, in the case where a developer may need to retain a small chunk of memory from a pool for an indeterminate amount of time, it may be appropriate to create an un-pooled `Buffer` instance using `Buffer.allocUnsafeSlow()` then copy out the relevant bits.

Exemplo:

```js
// Precisa manter em torno de alguns pequenos pedaços de memória
const store = [];

socket.on('readable', () => {
  const data = socket.read();

  // Alocar para dados retidos
  const sb = Buffer.allocUnsafeSlow(10);

  // Copie os dados para a nova alocação
  data.copy(sb, 0, 0, 10);

  store.push(sb);
});
```

Use of `Buffer.allocUnsafeSlow()` should be used only as a last resort *after* a developer has observed undue memory retention in their applications.

Um `TypeError` será lançado se `size` não é um número.

### Método de Classe: Buffer.byteLength(string[, encoding])

<!-- YAML
added: v0.1.90
changes:

  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/8946
    description: Passing invalid input will now throw an error.
  - version: v5.10.0
    pr-url: https://github.com/nodejs/node/pull/5255
    description: The `string` parameter can now be any `TypedArray`, `DataView`
                 or `ArrayBuffer`.
-->

* `string` {string|Buffer|TypedArray|DataView|ArrayBuffer|SharedArrayBuffer} Um valor para calcular o comprimento de.
* `encoding` {string} Se `string` é uma string, esta é sua codificação. **Default:** `'utf8'`.
* Retorna: {integer} O número de bytes contidos dentro de `string`.

Retorna o tamanho atual do byte de uma string. Isto não é o mesmo que [`String.prototype.length`] uma vez que retorna o número de *caracteres* em uma string.

*Note*: For `'base64'` and `'hex'`, this function assumes valid input. For strings that contain non-Base64/Hex-encoded data (e.g. whitespace), the return value might be greater than the length of a `Buffer` created from the string.

Exemplo:

```js
const str = '\u00bd + \u00bc = \u00be';

// Prints: ½ + ¼ = ¾: 9 characters, 12 bytes
console.log(`${str}: ${str.length} characters, ` +
            `${Buffer.byteLength(str, 'utf8')} bytes`);
```

Quando `string` é um `Buffer`/[`DataView`]/[`TypedArray`]/[`ArrayBuffer`]/ [`SharedArrayBuffer`], o tamanho do byte real é devolvido.

### Método de Classe: Buffer.compare(buf1, buf2)

<!-- YAML
added: v0.11.13
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The arguments can now be `Uint8Array`s.
-->

* `buf1` {Buffer|Uint8Array}
* `buf2` {Buffer|Uint8Array}
* Retorna: {integer}

Compara `buf1` com `buf2` normalmente para o propósito de ordenar os arrays de instâncias `Buffer`. Isto é equivalente a chamar [`buf1.compare(buf2)`][`buf.compare()`].

Exemplo:

```js
const buf1 = Buffer.from('1234');
const buf2 = Buffer.from('0123');
const arr = [buf1, buf2];

// Prints: [ <Buffer 30 31 32 33>, <Buffer 31 32 33 34> ]
// (This result is equal to: [buf2, buf1])
console.log(arr.sort(Buffer.compare));
```

### Método de Classe: Buffer.concat(list[, totalLength])

<!-- YAML
added: v0.7.11
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The elements of `list` can now be `Uint8Array`s.
-->

* `list` {Array} List of `Buffer` or [`Uint8Array`] instances to concat.
* `totalLength` {integer} Comprimento total das instâncias `Buffer` em `list` quando concatenado.
* Retorna: {Buffer}

Retorna um novo `Buffer` que é o resultado de concatenar todos as instâncias `Buffer` em `list` juntos.

Se a lista não tem itens, ou se o `totalLength` é 0, então um novo `Buffer` de comprimento zero é retornado.

Se `totalLength` não for fornecido, ele é calculado a partir das instâncias `Buffer` em `list`. Isso, no entanto, faz com que um loop adicional seja executado para calcular o `totalLength`, então é mais rápido fornecer o comprimento explicitamente se já é conhecido.

Se `totalLength` for fornecido, ele é coagido a um inteiro não assinado. Se o comprimento combinado dos `Buffer`s na `list` excede `totalLength`, o resultado é truncado para `totalLength`.

Example: Create a single `Buffer` from a list of three `Buffer` instances

```js
const buf1 = Buffer.alloc(10);
const buf2 = Buffer.alloc(14);
const buf3 = Buffer.alloc(18);
const totalLength = buf1.length + buf2.length + buf3.length;

// Prints: 42
console.log(totalLength);

const bufA = Buffer.concat([buf1, buf2, buf3], totalLength);

// Prints: <Buffer 00 00 00 00 ...>
console.log(bufA);

// Prints: 42
console.log(bufA.length);
```

### Método de Classe: Buffer.from(array)

<!-- YAML
added: v5.10.0
-->

* `array` {Array}

Aloca um novo `Buffer` usando um `array` de octets.

Exemplo:

```js
// Cria um novo Buffer contendo UTF-8 bytes of the string 'buffer'
const buf = Buffer.from([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

Um `TypeError` será lançado se `array` não é um `Array`.

### Método de Classe: Buffer.from(arrayBuffer[, byteOffset[, length]])

<!-- YAML
added: v5.10.0
-->

* `arrayBuffer` {ArrayBuffer|SharedArrayBuffer} Um [`ArrayBuffer`], [`SharedArrayBuffer`], ou a propriedade `.buffer` de um [`TypedArray`].
* `byteOffset` {integer} Index do primeiro byte para expor. **Default:** `0`.
* `length` {integer} Número de bytes para expor. **Default:** `arrayBuffer.length - byteOffset`.

Isso cria uma visão do [`ArrayBuffer`] sem copiar a memória subjacente. Por exemplo, quando passar uma referência à propriedade `.buffer` de um [`TypedArray`] a nova instância criada `Buffer` compartilhará a mesma memória alocada como a [`TypedArray`].

Exemplo:

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`
const buf = Buffer.from(arr.buffer);

// Prints: <Buffer 88 13 a0 0f>
console.log(buf);

// Changing the original Uint16Array changes the Buffer also
arr[1] = 6000;

// Prints: <Buffer 88 13 70 17>
console.log(buf);
```

Os argumentos opcionais `byteOffset` e `length` especificam um intervalo de memória dentro de `arrayBuffer` que será compartilhado pelo `Buffer`.

Exemplo:

```js
const ab = new ArrayBuffer(10);
const buf = Buffer.from(ab, 0, 2);

// Prints: 2
console.log(buf.length);
```

Um `TypeError` será jogado se `arrayBuffer` não é um [`ArrayBuffer`] ou um [`SharedArrayBuffer`].

### Método de Classe: Buffer.from(buffer)

<!-- YAML
added: v5.10.0
-->

* `buffer` {Buffer|Uint8Array} An existing `Buffer` or [`Uint8Array`] from which to copy data.

Copia os dados passados `buffer` em uma nova instância `Buffer`.

Exemplo:

```js
const buf1 = Buffer.from('buffer');
const buf2 = Buffer.from(buf1);

buf1[0] = 0x61;

// Prints: auffer
console.log(buf1.toString());

// Prints: buffer
console.log(buf2.toString());
```

Um `TypeError` será lançado se `buffer` não é um `Buffer`.

### Método de Classe: Buffer.from(string[, encoding])

<!-- YAML
added: v5.10.0
-->

* `string` {string} Uma string para codificar.
* `encoding` {string} A codificação da `string`. **Default:** `'utf8'`.

Cria uma nova `Buffer` que contém `string`. The `encoding` parameter identifies the character encoding of `string`.

Examples:

```js
const buf1 = Buffer.from('this is a tést');

// Prints: this is a tést
console.log(buf1.toString());

// Prints: this is a tC)st
console.log(buf1.toString('ascii'));

const buf2 = Buffer.from('7468697320697320612074c3a97374', 'hex');

// Prints: this is a tést
console.log(buf2.toString());
```

Um `TypeError` será lançado se `string` não for uma string.

### Método de Classe: Buffer.from(object[, offsetOrEncoding[, length]])

<!-- YAML
added: v8.2.0
-->

* `objeto` {Object} Um objeto que suporta `Symbol.toPrimitive` ou `valueOf()`
* `offsetOrEncoding` {number|string} A byte-offset or encoding, depending on the value returned either by `object.valueOf()` or `object[Symbol.toPrimitive]()`.
* `length` {number} A length, depending on the value returned either by `object.valueOf()` or `object[Symbol.toPrimitive]()`.

For objects whose `valueOf()` function returns a value not strictly equal to `object`, returns `Buffer.from(object.valueOf(), offsetOrEncoding, length)`.

Por exemplo:

```js
const buf = Buffer.from(new String('this is a test'));
// <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

For objects that support `Symbol.toPrimitive`, returns `Buffer.from(object[Symbol.toPrimitive](), offsetOrEncoding, length)`.

Por exemplo:

```js
class Foo {
  [Symbol.toPrimitive]() {
    return 'this is a test';
  }
}

const buf = Buffer.from(new Foo(), 'utf8');
// <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

### Método de Classe: Buffer.isBuffer(obj)

<!-- YAML
added: v0.1.101
-->

* `obj` {Object}
* Retorna: {boolean}

Retorna `true` se `obj` é um `Buffer`, `false` caso contrário.

### Método de Classe: Buffer.isEncoding(encoding)

<!-- YAML
added: v0.9.1
-->

* `encoding` {string} Um nome de codificação de caracteres para verificar.
* Retorna: {boolean}

Retorna `true` se `encoding` contém uma codificação de caracteres suportada, ou `false` caso contrário.

### Propriedade da classe: Buffer.poolSize

<!-- YAML
added: v0.11.3
-->

* {integer} **Default:** `8192`

This is the number of bytes used to determine the size of pre-allocated, internal `Buffer` instances used for pooling. Este valor pode ser modificado.

### buf[index]

<!-- YAML
type: property
name: [index]
-->

O operador de índice `[index]` pode ser usado para obter e definir o octet na posição `index` em `buf`. Os valores referem-se a bytes individuais, então o valor legal tem entre `0x00` e `0xFF` (hex) ou `0` e `255` (decimal).

Este operador é herdado de `Uint8Array`, então seu comportamento em acesso fora dos limites é o mesmo que `UInt8Array` - ou seja, retorna `undefined` e a configuração não faz nada.

Example: Copy an ASCII string into a `Buffer`, one byte at a time

```js
const str = 'Node.js';
const buf = Buffer.allocUnsafe(str.length);

for (let i = 0; i < str.length; i++) {
  buf[i] = str.charCodeAt(i);
}

// Prints: Node.js
console.log(buf.toString('ascii'));
```

### buf.buffer

The `buffer` property references the underlying `ArrayBuffer` object based on which this Buffer object is created.

```js
const arrayBuffer = new ArrayBuffer(16);
const buffer = Buffer.from(arrayBuffer);

console.log(buffer.buffer === arrayBuffer);
// Imprime: true
```

### buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]])

<!-- YAML
added: v0.11.13
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `target` parameter can now be a `Uint8Array`.
  - version: v5.11.0
    pr-url: https://github.com/nodejs/node/pull/5880
    description: Additional parameters for specifying offsets are supported now.
-->

* `target` {Buffer|Uint8Array} A `Buffer` or [`Uint8Array`] to compare to.
* `targetStart` {integer} O offset de `target` com o qual começa a comparação. **Default:** `0`.
* `targetEnd` {integer} O offset de `target` com o qual termina a comparação. (Não incluída). **Default:** `target.length`.
* `sourceStart` {integer} O offset de `buf` com o qual começa a comparação. **Default:** `0`.
* `sourceEnd` {integer} O offset de `buf` com o qual termina a comparação. (Não incluída). **Default:** [`buf.length`].
* Retorna: {integer}

Compara `buf` com `target` e retorna um número indicando se `buf` vem antes, depois, ou é o mesmo que `target` em ordem de ordenação. Comparação é baseada na sequência real de bytes em cada `Buffer`.

* `0` é retornado se `target` é o mesmo que `buf`
* `1` é retornado se `target` deve vir *antes* de `buf` quando ordenado.
* `-1` é retornado se `target` deve vir *depois* de `buf` quando ordenado.

Examples:

```js
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('BCD');
const buf3 = Buffer.from('ABCD');

// Prints: 0
console.log(buf1.compare(buf1));

// Prints: -1
console.log(buf1.compare(buf2));

// Prints: -1
console.log(buf1.compare(buf3));

// Prints: 1
console.log(buf2.compare(buf1));

// Prints: 1
console.log(buf2.compare(buf3));

// Prints: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
// (This result is equal to: [buf1, buf3, buf2])
console.log([buf1, buf2, buf3].sort(Buffer.compare));
```

Os argumentos opcionais `targetStart`, `targetEnd`, `sourceStart`, e `sourceEnd` podem ser usados para limitar a comparação a intervalos específicos dentro de `target` e `buf` respectivamente.

Examples:

```js
const buf1 = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const buf2 = Buffer.from([5, 6, 7, 8, 9, 1, 2, 3, 4]);

// Prints: 0
console.log(buf1.compare(buf2, 5, 9, 0, 4));

// Prints: -1
console.log(buf1.compare(buf2, 0, 6, 4));

// Prints: 1
console.log(buf1.compare(buf2, 5, 6, 5));
```

A `RangeError` will be thrown if: `targetStart < 0`, `sourceStart < 0`, `targetEnd > target.byteLength` or `sourceEnd > source.byteLength`.

### buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])

<!-- YAML
added: v0.1.90
-->

* `target` {Buffer|Uint8Array} Um `Buffer` ou [`Uint8Array`] para copiar em.
* `targetStart` {integer} The offset within `target` at which to begin copying to. **Default:** `0`.
* `sourceStart` {integer} The offset within `buf` at which to begin copying from. **Default:** `0`.
* `sourceEnd` {integer} O offset de `buf` com o qual termina de copiar. (Não incluída). **Default:** [`buf.length`].
* Retorna: {integer} O número de bytes copiados.

Copia dados de uma região de `buf` para uma região em `target` mesmo se a região da memória de `target` se sobrepõe com `buf`.

Example: Create two `Buffer` instances, `buf1` and `buf2`, and copy `buf1` from byte 16 through byte 19 into `buf2`, starting at the 8th byte in `buf2`

```js
const buf1 = Buffer.allocUnsafe(26);
const buf2 = Buffer.allocUnsafe(26).fill('!');

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf1[i] = i + 97;
}

buf1.copy(buf2, 8, 16, 20);

// Prints: !!!!!!!!qrst!!!!!!!!!!!!!
console.log(buf2.toString('ascii', 0, 25));
```

Example: Create a single `Buffer` and copy data from one region to an overlapping region within the same `Buffer`

```js
const buf = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf[i] = i + 97;
}

buf.copy(buf, 0, 4, 10);

// Prints: efghijghijklmnopqrstuvwxyz
console.log(buf.toString());
```

### buf.entries()

<!-- YAML
added: v1.1.0
-->

* Retorna: {Iterator}

Cria e retorna um [iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) de `[index, byte]` pares do conteúdo de `buf`.

Example: Log the entire contents of a `Buffer`

```js
const buf = Buffer.from('buffer');

// Prints:
//   [0, 98]
//   [1, 117]
//   [2, 102]
//   [3, 102]
//   [4, 101]
//   [5, 114]
for (const pair of buf.entries()) {
  console.log(pair);
}
```

### buf.equals(otherBuffer)

<!-- YAML
added: v0.11.13
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The arguments can now be `Uint8Array`s.
-->

* `otherBuffer` {Buffer} A `Buffer` or [`Uint8Array`] to compare to.
* Retorna: {boolean}

Retorna `true` se ambos `buf` e `otherBuffer` tiverem exatamente os mesmos bytes, `false` caso contrário.

Examples:

```js
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('414243', 'hex');
const buf3 = Buffer.from('ABCD');

// Prints: true
console.log(buf1.equals(buf2));

// Prints: false
console.log(buf1.equals(buf3));
```

### buf.fill(value\[, offset[, end]\]\[, encoding\])

<!-- YAML
added: v0.5.0
changes:

  - version: v5.7.0
    pr-url: https://github.com/nodejs/node/pull/4935
    description: The `encoding` parameter is supported now.
-->

* `value` {string|Buffer|integer} The value to fill `buf` with.
* `offset` {integer} Número de bytes para pular antes de começar a preencher `buf`. **Default:** `0`.
* `end` {integer} Onde para de preencher `buf` (não incluso). **Default:** [`buf.length`].
* `encoding` {string} Se `value` é uma string, esta é sua codificação. **Default:** `'utf8'`.
* Retorna: {Buffer} Uma referência a `buf`.

Preenche `buf` com o valor especificado `value`. If the `offset` and `end` are not given, the entire `buf` will be filled. This is meant to be a small simplification to allow the creation and filling of a `Buffer` to be done on a single line.

Example: Fill a `Buffer` with the ASCII character `'h'`

```js
const b = Buffer.allocUnsafe(50).fill('h');

// Prints: hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
console.log(b.toString());
```

`value` is coerced to a `uint32` value if it is not a String or Integer.

If the final write of a `fill()` operation falls on a multi-byte character, then only the first bytes of that character that fit into `buf` are written.

Example: Fill a `Buffer` with a two-byte character

```js
// Prints: <Buffer c8 a2 c8>
console.log(Buffer.allocUnsafe(3).fill('\u0222'));
```

If `value` contains invalid characters, it is truncated.

If no valid fill data remains, then the buffer is either zero-filled or no filling is performed, depending on the input type. That behavior is dictated by compatibility reasons and was changed to throwing an exception in Node.js v10, so it's not recommended to rely on that.

```js
const buf = Buffer.allocUnsafe(5);
// Prints: <Buffer 61 61 61 61 61>
console.log(buf.fill('a'));
// Prints: <Buffer aa aa aa aa aa>
console.log(buf.fill('aazz', 'hex'));
// Prints: <Buffer aa aa aa aa aa>
console.log(buf.fill('zz', 'hex'));
```

### buf.includes(value\[, byteOffset\]\[, encoding\])

<!-- YAML
added: v5.3.0
-->

* `value` {string|Buffer|integer} O que procurar.
* `byteOffset` {integer} Onde começar a procurar em `buf`. **Default:** `0`.
* `encoding` {string} Se `value` é uma string, esta é sua codificação. **Default:** `'utf8'`.
* Retorna: {boolean} `true` se `value` foi encontrado em `buf`, `false` caso contrário.

Equivalente para [`buf.indexOf() != -1`][`buf.indexOf()`].

Examples:

```js
const buf = Buffer.from('this is a buffer');

// Prints: true
console.log(buf.includes('this'));

// Prints: true
console.log(buf.includes('is'));

// Prints: true
console.log(buf.includes(Buffer.from('a buffer')));

// Prints: true
// (97 is the decimal ASCII value for 'a')
console.log(buf.includes(97));

// Prints: false
console.log(buf.includes(Buffer.from('a buffer example')));

// Prints: true
console.log(buf.includes(Buffer.from('a buffer example').slice(0, 8)));

// Prints: false
console.log(buf.includes('this', 4));
```

### buf.indexOf(value\[, byteOffset\]\[, encoding\])

<!-- YAML
added: v1.5.0
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `value` can now be a `Uint8Array`.
  - version: v5.7.0, v4.4.0
    pr-url: https://github.com/nodejs/node/pull/4803
    description: When `encoding` is being passed, the `byteOffset` parameter
                 is no longer required.
-->

* `value` {string|Buffer|Uint8Array|integer} O que procurar.
* `byteOffset` {integer} Onde começar a procurar em `buf`. **Default:** `0`.
* `encoding` {string} If `value` is a string, this is the encoding used to determine the binary representation of the string that will be searched for in `buf`. **Default:** `'utf8'`.
* Returns: {integer} The index of the first occurrence of `value` in `buf`, or `-1` if `buf` does not contain `value`.

Se `value` é:

* uma string, `value` é interpretada de acordo com a codificação de caracteres em `encoding`.
* um `Buffer` ou [`Uint8Array`], `value` será usado na sua totalidade. Para comparar um `Buffer` parcial, use [`buf.slice()`].
* um número, `value` será interpretado como um valor inteiro de 8 bits não assinado entre `0` e `255`.

Examples:

```js
const buf = Buffer.from('this is a buffer');

// Prints: 0
console.log(buf.indexOf('this'));

// Prints: 2
console.log(buf.indexOf('is'));

// Prints: 8
console.log(buf.indexOf(Buffer.from('a buffer')));

// Prints: 8
// (97 is the decimal ASCII value for 'a')
console.log(buf.indexOf(97));

// Prints: -1
console.log(buf.indexOf(Buffer.from('a buffer example')));

// Prints: 8
console.log(buf.indexOf(Buffer.from('a buffer example').slice(0, 8)));

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'ucs2');

// Prints: 4
console.log(utf16Buffer.indexOf('\u03a3', 0, 'ucs2'));

// Prints: 6
console.log(utf16Buffer.indexOf('\u03a3', -4, 'ucs2'));
```

Se `value` não é uma string, número, ou `Buffer`, este método irá lançar um `TypeError`. Se `value` é um número, ele será coagido a um valor de byte válido, um inteiro entre 0 e 255.

Se `byteOffset` não for um número, ele será coagido a um número. Any arguments that coerce to `NaN` or 0, like `{}`, `[]`, `null` or `undefined`, will search the whole buffer. This behavior matches [`String#indexOf()`].

```js
const b = Buffer.from('abcdef');

// Passando um valor que é um número, mas não um byte válido
// Imprime: 2, equivalente a procurar por 99 ou 'c'
console.log(b.indexOf(99.9));
console.log(b.indexOf(256 + 99));

// Passando um byteOffset que coage para NaN ou 0
// Imprime: 1, pesquisando todo o buffer
console.log(b.indexOf('b', undefined));
console.log(b.indexOf('b', {}));
console.log(b.indexOf('b', null));
console.log(b.indexOf('b', []));
```

If `value` is an empty string or empty `Buffer` and `byteOffset` is less than `buf.length`, `byteOffset` will be returned. If `value` is empty and `byteOffset` is at least `buf.length`, `buf.length` will be returned.

### buf.keys()

<!-- YAML
added: v1.1.0
-->

* Retorna: {Iterator}

Cria e retorna um [iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) de chaves `buf` (indices).

Exemplo:

```js
const buf = Buffer.from('buffer');

// Prints:
//   0
//   1
//   2
//   3
//   4
//   5
for (const key of buf.keys()) {
  console.log(key);
}
```

### buf.lastIndexOf(value\[, byteOffset\]\[, encoding\])

<!-- YAML
added: v6.0.0
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `value` can now be a `Uint8Array`.
-->

* `value` {string|Buffer|Uint8Array|integer} O que procurar.
* `byteOffset` {integer} Onde começar a procurar em `buf`. **Default:** [`buf.length`]`- 1`.
* `encoding` {string} If `value` is a string, this is the encoding used to determine the binary representation of the string that will be searched for in `buf`. **Default:** `'utf8'`.
* Returns: {integer} The index of the last occurrence of `value` in `buf`, or `-1` if `buf` does not contain `value`.

Identical to [`buf.indexOf()`], except the last occurrence of `value` is found rather than the first occurrence.

Examples:

```js
const buf = Buffer.from('this buffer is a buffer');

// Prints: 0
console.log(buf.lastIndexOf('this'));

// Prints: 17
console.log(buf.lastIndexOf('buffer'));

// Prints: 17
console.log(buf.lastIndexOf(Buffer.from('buffer')));

// Prints: 15
// (97 is the decimal ASCII value for 'a')
console.log(buf.lastIndexOf(97));

// Prints: -1
console.log(buf.lastIndexOf(Buffer.from('yolo')));

// Prints: 5
console.log(buf.lastIndexOf('buffer', 5));

// Prints: -1
console.log(buf.lastIndexOf('buffer', 4));

const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'ucs2');

// Prints: 6
console.log(utf16Buffer.lastIndexOf('\u03a3', undefined, 'ucs2'));

// Prints: 4
console.log(utf16Buffer.lastIndexOf('\u03a3', -5, 'ucs2'));
```

Se `value` não é uma string, número, ou `Buffer`, este método irá lançar um `TypeError`. Se `value` é um número, ele será coagido a um valor de byte válido, um inteiro entre 0 e 255.

Se `byteOffset` não for um número, ele será coagido a um número. Qualquer argumento que coagir com `NaN`, como `{}` ou `undefined`, irá pesquisar todo o buffer. Este comportamento corresponde a [`String#lastIndexOf()`].

```js
const b = Buffer.from('abcdef');

// Passando um valor que é um número, mas não um byte válido
// Imprime: 2, equivalente a procurar por 99 ou 'c'
console.log(b.lastIndexOf(99.9));
console.log(b.lastIndexOf(256 + 99));

// Passando um byteOffset que coage para NaN
// Imprime: 1, pesquisando todo o buffer
console.log(b.lastIndexOf('b', undefined));
console.log(b.lastIndexOf('b', {}));

// Passando um byteOffset que coage para 0
// Imprime: -1, equivalente a passar 0
console.log(b.lastIndexOf('b', null));
console.log(b.lastIndexOf('b', []));
```

Se `value` é uma string vazia ou um `Buffer`, vazio `byteOffset` será devolvido.

### buf.length

<!-- YAML
added: v0.1.90
-->

* {integer}

Retorna a quantidade de memória alocada para `buf` em bytes. Note que este não reflete necessariamente a quantidade de dados "utilizáveis" dentro de `buf`.

Example: Create a `Buffer` and write a shorter ASCII string to it

```js
const buf = Buffer.alloc(1234);

// Prints: 1234
console.log(buf.length);

buf.write('some string', 0, 'ascii');

// Prints: 1234
console.log(buf.length);
```

Enquanto a propriedade `length` não é imutável, mudando o valor de `length` pode resultar em comportamento indefinido e inconsistente. Aplicações que desejam modificar o comprimento de um `Buffer` devem, portanto, tratar `length` como somente leitura e usar [`buf.slice()`] para criar um novo `Buffer`.

Examples:

```js
let buf = Buffer.allocUnsafe(10);

buf.write('abcdefghj', 0, 'ascii');

// Prints: 10
console.log(buf.length);

buf = buf.slice(0, 5);

// Prints: 5
console.log(buf.length);
```

### buf.parent

<!-- YAML
deprecated: v8.0.0
-->

> Estabilidade: 0 - Descontinuada: Use [`buf.buffer`] em vez disso.

A propriedade `buf.parent` é um alias obsoleto para `buf.buffer`.

### buf.readDoubleBE(offset[, noAssert])

### buf.readDoubleLE(offset[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 8`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {number}

Lê um duplo de 64 bits de `buf` no `offset` especificado com formato especificado endian (`readDoubleBE()` retorna grande endian, `readbleLE()` retorna pequeno endian).

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

// Prints: 8.20788039913184e-304
console.log(buf.readDoubleBE());

// Prints: 5.447603722011605e-270
console.log(buf.readDoubleLE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readDoubleLE(1));

// Warning: reads passed end of buffer!
// This will result in a segmentation fault! Don't do this!
console.log(buf.readDoubleLE(1, true));
```

### buf.readFloatBE(offset[, noAssert])

### buf.readFloatLE(offset[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 4`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {number}

Lê um float de 32 bits de `buf` no especificado `offset` com formato especificado endian (`readFloatBE()` retorna grande endian, `readatLE()` retorna pequeno endian).

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([1, 2, 3, 4]);

// Prints: 2.387939260590663e-38
console.log(buf.readFloatBE());

// Prints: 1.539989614439558e-36
console.log(buf.readFloatLE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readFloatLE(1));

// Warning: reads passed end of buffer!
// This will result in a segmentation fault! Don't do this!
console.log(buf.readFloatLE(1, true));
```

### buf.readInt8(offset[, noAssert])

<!-- YAML
added: v0.5.0
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 1`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {integer}

Lê um 8 bits assinado inteiro de `buf` no `offset` especificado.

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Inteiros lidos de um `Buffer` são interpretados como dois valores assinados complementares.

Examples:

```js
const buf = Buffer.from([-1, 5]);

// Prints: -1
console.log(buf.readInt8(0));

// Prints: 5
console.log(buf.readInt8(1));

// Throws an exception: RangeError: Index out of range
console.log(buf.readInt8(2));
```

### buf.readInt16BE(offset[, noAssert])

### buf.readInt16LE(offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 2`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {integer}

Lê um inteiro de 16 bits assinado de `buf` no `offset` especificado com o formato endian selecionado (`readInt16BE()` retorna grande endian, `readInt16LE()` retorna pequeno endian).

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Inteiros lidos de um `Buffer` são interpretados como dois valores assinados complementares.

Examples:

```js
const buf = Buffer.from([0, 5]);

// Prints: 5
console.log(buf.readInt16BE());

// Prints: 1280
console.log(buf.readInt16LE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readInt16LE(1));
```

### buf.readInt32BE(offset[, noAssert])

### buf.readInt32LE(offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 4`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {integer}

Lê um inteiro de 32 bits assinado de `buf` no `offset` especificado com o formato selecionado (`readInt32BE()` retorna grande endian, `readInt32LE()` retorna pequeno endian).

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Inteiros lidos de um `Buffer` são interpretados como dois valores assinados complementares.

Examples:

```js
const buf = Buffer.from([0, 0, 0, 5]);

// Prints: 5
console.log(buf.readInt32BE());

// Prints: 83886080
console.log(buf.readInt32LE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readInt32LE(1));
```

### buf.readIntBE(offset, byteLength[, noAssert])

### buf.readIntLE(offset, byteLength[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - byteLength`.
* `byteLength` {integer} Número de bytes para ler. Must satisfy: `0 < byteLength <= 6`.
* `noAssert` {boolean} Skip `offset` and `byteLength` validation? **Default:** `false`.
* Retorna: {integer}

Lê `byteLength` número de bytes de `buf` no `offset` especificado e interpreta o resultado como dois valores complementares assinalados. Suporta até 48 bits de precisão.

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

// Prints: -546f87a9cbee
console.log(buf.readIntLE(0, 6).toString(16));

// Prints: 1234567890ab
console.log(buf.readIntBE(0, 6).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readIntBE(1, 6).toString(16));
```

### buf.readUInt8(offset[, noAssert])

<!-- YAML
added: v0.5.0
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 1`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {integer}

Lê um inteiro de 8 bits não assinado de `buf` no `offset` especificado.

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([1, -2]);

// Prints: 1
console.log(buf.readUInt8(0));

// Prints: 254
console.log(buf.readUInt8(1));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUInt8(2));
```

### buf.readUInt16BE(offset[, noAssert])

### buf.readUInt16LE(offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 2`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {integer}

Lê um inteiro de 16 bits não assinado de `buf` no `offset` especificado com Formato endian especificado (`readUInt16BE()` retorna grande endian, `readUInt16LE()` retorna pequeno endian).

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([0x12, 0x34, 0x56]);

// Prints: 1234
console.log(buf.readUInt16BE(0).toString(16));

// Prints: 3412
console.log(buf.readUInt16LE(0).toString(16));

// Prints: 3456
console.log(buf.readUInt16BE(1).toString(16));

// Prints: 5634
console.log(buf.readUInt16LE(1).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUInt16LE(2).toString(16));
```

### buf.readUInt32BE(offset[, noAssert])

### buf.readUInt32LE(offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - 4`.
* `noAssert` {boolean} Skip `offset` validation? **Default:** `false`
* Retorna: {integer}

Lê um inteiro de 32 bits não assinado de `buf` no `offset` especificado com Formato endian especificado (`readUInt32BE()` retorna grande endian, `readUInt32LE()` retorna pequeno endian).

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

// Prints: 12345678
console.log(buf.readUInt32BE(0).toString(16));

// Prints: 78563412
console.log(buf.readUInt32LE(0).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUInt32LE(1).toString(16));
```

### buf.readUIntBE(offset, byteLength[, noAssert])

### buf.readUIntLE(offset, byteLength[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `offset` {integer} Número de bytes para pular antes de começar a ler. Must satisfy: `0 <= offset <= buf.length - byteLength`.
* `byteLength` {integer} Número de bytes para ler. Must satisfy: `0 < byteLength <= 6`.
* `noAssert` {boolean} Skip `offset` and `byteLength` validation? **Default:** `false`
* Retorna: {integer}

Lê `byteLength` número de bytes de `buf` no `offset` especificado e interpreta o resultado como um inteiro não assinado. Suporta até 48 bits de precisão.

Setting `noAssert` to `true` allows `offset` to be beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

// Prints: 1234567890ab
console.log(buf.readUIntBE(0, 6).toString(16));

// Prints: ab9078563412
console.log(buf.readUIntLE(0, 6).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUIntBE(1, 6).toString(16));
```

### buf.slice([start[, end]])

<!-- YAML
added: v0.3.0
changes:

  - version: v7.1.0, v6.9.2
    pr-url: https://github.com/nodejs/node/pull/9341
    description: Coercing the offsets to integers now handles values outside
                 the 32-bit integer range properly.
  - version: v7.0.0
    pr-url: https://github.com/nodejs/node/pull/9101
    description: All offsets are now coerced to integers before doing any
                 calculations with them.
-->

* `start` {integer} Onde o novo `Buffer` começará. **Default:** `0`.
* `end` {integer} Onde o novo `Buffer` terminará (não inclusivo). **Default:** [`buf.length`].
* Retorna: {Buffer}

Retorna um novo `Buffer` que faz referência à mesma memória que o original, mas offset e cortado pelos índices `start` e `end`.

Specifying `end` greater than [`buf.length`] will return the same result as that of `end` equal to [`buf.length`].

*Note*: Modifying the new `Buffer` slice will modify the memory in the original `Buffer` because the allocated memory of the two objects overlap.

Example: Create a `Buffer` with the ASCII alphabet, take a slice, and then modify one byte from the original `Buffer`

```js
const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf1[i] = i + 97;
}

const buf2 = buf1.slice(0, 3);

// Prints: abc
console.log(buf2.toString('ascii', 0, buf2.length));

buf1[0] = 33;

// Prints: !bc
console.log(buf2.toString('ascii', 0, buf2.length));
```

Especificar índices negativos faz com que a fatia seja gerada em relação ao fim de `buf` em vez do início.

Examples:

```js
const buf = Buffer.from('buffer');

// Prints: buffe
// (Equivalent to buf.slice(0, 5))
console.log(buf.slice(-6, -1).toString());

// Prints: buff
// (Equivalent to buf.slice(0, 4))
console.log(buf.slice(-6, -2).toString());

// Prints: uff
// (Equivalent to buf.slice(1, 4))
console.log(buf.slice(-5, -2).toString());
```

### buf.swap16()

<!-- YAML
added: v5.10.0
-->

* Retorna: {Buffer} Uma referência a `buf`.

Interprets `buf` as an array of unsigned 16-bit integers and swaps the byte order *in-place*. Throws a `RangeError` if [`buf.length`] is not a multiple of 2.

Examples:

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

// Prints: <Buffer 01 02 03 04 05 06 07 08>
console.log(buf1);

buf1.swap16();

// Prints: <Buffer 02 01 04 03 06 05 08 07>
console.log(buf1);

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

// Throws an exception: RangeError: Buffer size must be a multiple of 16-bits
buf2.swap16();
```

### buf.swap32()

<!-- YAML
added: v5.10.0
-->

* Retorna: {Buffer} Uma referência a `buf`.

Interprets `buf` as an array of unsigned 32-bit integers and swaps the byte order *in-place*. Throws a `RangeError` if [`buf.length`] is not a multiple of 4.

Examples:

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

// Prints: <Buffer 01 02 03 04 05 06 07 08>
console.log(buf1);

buf1.swap32();

// Prints: <Buffer 04 03 02 01 08 07 06 05>
console.log(buf1);

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

// Throws an exception: RangeError: Buffer size must be a multiple of 32-bits
buf2.swap32();
```

### buf.swap64()

<!-- YAML
added: v6.3.0
-->

* Retorna: {Buffer} Uma referência a `buf`.

Interprets `buf` as an array of 64-bit numbers and swaps the byte order *in-place*. Throws a `RangeError` if [`buf.length`] is not a multiple of 8.

Examples:

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

// Prints: <Buffer 01 02 03 04 05 06 07 08>
console.log(buf1);

buf1.swap64();

// Prints: <Buffer 08 07 06 05 04 03 02 01>
console.log(buf1);

const buf2 = Buffer.from([0x1, 0x2, 0x3]);

// Throws an exception: RangeError: Buffer size must be a multiple of 64-bits
buf2.swap64();
```

Note que o JavaScript não pode codificar números inteiros de 64 bits. Este método é destinado para trabalhar com floats 64 bits.

### buf.toJSON()

<!-- YAML
added: v0.9.2
-->

* Retorna: {Object}

Retorna uma representação JSON de `buf`. [`JSON.stringify()`] chama implicitamente esta função quando stringifying uma instância `Buffer`.

Exemplo:

```js
const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

// Prints: {"type":"Buffer","data":[1,2,3,4,5]}
console.log(json);

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value.data) :
    value;
});

// Prints: <Buffer 01 02 03 04 05>
console.log(copy);
```

### buf.toString([encoding[, start[, end]]])

<!-- YAML
added: v0.1.90
-->

* `encoding` {string} The character encoding to decode to. **Default:** `'utf8'`.
* `start` {integer} O offset de bytes para começar a decodificar. **Default:** `0`.
* `end` {integer} O offset de bytes para parar de decodificar em (não incluído). **Default:** [`buf.length`].
* Retorna: {string}

Decodifica `buf` para uma string de acordo com a codificação de caracteres especificada em `encoding`. `start` e `end` podem ser passados para decodificar apenas um subconjunto de `buf`.

The maximum length of a string instance (in UTF-16 code units) is available as [`buffer.constants.MAX_STRING_LENGTH`][].

Examples:

```js
const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf1[i] = i + 97;
}

// Prints: abcdefghijklmnopqrstuvwxyz
console.log(buf1.toString('ascii'));

// Prints: abcde
console.log(buf1.toString('ascii', 0, 5));

const buf2 = Buffer.from('tést');

// Prints: 74c3a97374
console.log(buf2.toString('hex'));

// Prints: té
console.log(buf2.toString('utf8', 0, 3));

// Prints: té
console.log(buf2.toString(undefined, 0, 3));
```

### buf.values()

<!-- YAML
added: v1.1.0
-->

* Retorna: {Iterator}

Cria e retorna um [iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) para valores `buf` (bytes). Esta função é chamada automaticamente quando um `Buffer` é usado em uma instrução `for..of`.

Examples:

```js
const buf = Buffer.from('buffer');

// Prints:
//   98
//   117
//   102
//   102
//   101
//   114
for (const value of buf.values()) {
  console.log(value);
}

// Prints:
//   98
//   117
//   102
//   102
//   101
//   114
for (const value of buf) {
  console.log(value);
}
```

### buf.write(string\[, offset[, length]\]\[, encoding\])

<!-- YAML
added: v0.1.90
-->

* `string` {string} String to be written to `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever a `string`. **Default:** `0`.
* `length` {integer} Número de bytes para escrever. **Default:** `buf.length - offset`.
* `encoding` {string} A codificação de caracteres da `string`. **Default:** `'utf8'`.
* Retorna: {integer} Número de bytes escritos.

Writes `string` to `buf` at `offset` according to the character encoding in `encoding`. O parâmetro `length` é o número de bytes a escrever. If `buf` did not contain enough space to fit the entire string, only a partial amount of `string` will be written. No entanto, caracteres parcialmente codificados não serão escritos.

Exemplo:

```js
const buf = Buffer.allocUnsafe(256);

const len = buf.write('\u00bd + \u00bc = \u00be', 0);

// Prints: 12 bytes: ½ + ¼ = ¾
console.log(`${len} bytes: ${buf.toString('utf8', 0, len)}`);
```

### buf.writeDoubleBE(value, offset[, noAssert])

### buf.writeDoubleLE(value, offset[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `value` {number} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 8`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` para `buf` no `offset` específico com formato especificado endian (`write eDoubleBE()` escreve grande endian, `write eDoubleLE()` escreve pequeno endian). `value` *deve* ser um duplo de 64 bits válido. Comportamento não está definido quando `value` é algo diferente de um duplo de 64 bits.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(8);

buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

// Prints: <Buffer 43 eb d5 b7 dd f9 5f d7>
console.log(buf);

buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

// Prints: <Buffer d7 5f f9 dd b7 d5 eb 43>
console.log(buf);
```

### buf.writeFloatBE(value, offset[, noAssert])

### buf.writeFloatLE(value, offset[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `value` {number} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 4`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` para `buf` no `offset` com formato especificado endian (`write eFloatBE()` escreve grande endian, `write eFloatLE()` escreve pequeno endian). `value` *deve* ser um float de 32 bits válido. Comportamento não está definido quando `value` é qualquer coisa diferente de um float de 32 bits.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(4);

buf.writeFloatBE(0xcafebabe, 0);

// Prints: <Buffer 4f 4a fe bb>
console.log(buf);

buf.writeFloatLE(0xcafebabe, 0);

// Prints: <Buffer bb fe 4a 4f>
console.log(buf);
```

### buf.writeInt8(value, offset[, noAssert])

<!-- YAML
added: v0.5.0
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 1`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` para `buf` no `offset` especificado. `value` *deve* ser um inteiro válido assinado de 8 bits. Comportamento não está definido quando `value` é qualquer coisa diferente de um inteiro de 8 bits assinado.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

`value` é interpretado e escrito como dois inteiros complementares assinados.

Examples:

```js
const buf = Buffer.allocUnsafe(2);

buf.writeInt8(2, 0);
buf.writeInt8(-2, 1);

// Prints: <Buffer 02 fe>
console.log(buf);
```

### buf.writeInt16BE(value, offset[, noAssert])

### buf.writeInt16LE(value, offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 2`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` para `buf` no `offset` com formato especificado endian (`writeInt16BE()` escreve grande endian, `writeInt16LE()` escreve pequeno endian). `value` *deve* ser um inteiro válido assinado de 16 bits. Behavior is undefined when `value` is anything other than a signed 16-bit integer.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

`value` é interpretado e escrito como dois inteiros complementares assinados.

Examples:

```js
const buf = Buffer.allocUnsafe(4);

buf.writeInt16BE(0x0102, 0);
buf.writeInt16LE(0x0304, 2);

// Prints: <Buffer 01 02 04 03>
console.log(buf);
```

### buf.writeInt32BE(value, offset[, noAssert])

### buf.writeInt32LE(value, offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 4`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` para `buf` no `offset` com formato especificado endian (`writeInt32BE()` escreve grande endian, `writeInt32LE()` escreve pequeno endian). `value` *deve* ser um inteiro válido assinado de 32 bits. Behavior is undefined when `value` is anything other than a signed 32-bit integer.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

`value` é interpretado e escrito como dois inteiros complementares assinados.

Examples:

```js
const buf = Buffer.allocUnsafe(8);

buf.writeInt32BE(0x01020304, 0);
buf.writeInt32LE(0x05060708, 4);

// Prints: <Buffer 01 02 03 04 08 07 06 05>
console.log(buf);
```

### buf.writeIntBE(value, offset, byteLength[, noAssert])

### buf.writeIntLE(value, offset, byteLength[, noAssert])

<!-- YAML
added: v0.11.15
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - byteLength`.
* `byteLength` {integer} Número de bytes para escrever. Must satisfy: `0 < byteLength <= 6`.
* `noAssert` {boolean} Skip `value`, `offset`, and `byteLength` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `byteLength` bytes de `value` para `buf` no `offset` especificado. Suporta até 48 bits de precisão. Comportamento não está definido quando `value` é qualquer coisa diferente de um inteiro assinado.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(6);

buf.writeIntBE(0x1234567890ab, 0, 6);

// Prints: <Buffer 12 34 56 78 90 ab>
console.log(buf);

buf.writeIntLE(0x1234567890ab, 0, 6);

// Prints: <Buffer ab 90 78 56 34 12>
console.log(buf);
```

### buf.writeUInt8(value, offset[, noAssert])

<!-- YAML
added: v0.5.0
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 1`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` para `buf` no `offset` especificado. `value` *deve* ser um inteiro válido não assinado de 8 bits. Comportamento não está definido quando `value` é qualquer coisa diferente de um inteiro de 8 bits não assinado.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

// Prints: <Buffer 03 04 23 42>
console.log(buf);
```

### buf.writeUInt16BE(value, offset[, noAssert])

### buf.writeUInt16LE(value, offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 2`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` no`buf` no `offset` especificado, com formato especificado endian (`writeInt16BE()` escreve em big endian, `writeInt16LE()` escreve em little endian). `value` deve ser um inteiro válido não assinado de 16 bits. O comportamento é indefinido quando `value` é qualquer coisa diferente de um inteiro de 16 bits não assinado.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

// Prints: <Buffer de ad be ef>
console.log(buf);

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

// Prints: <Buffer ad de ef be>
console.log(buf);
```

### buf.writeUInt32BE(value, offset[, noAssert])

### buf.writeUInt32LE(value, offset[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - 4`.
* `noAssert` {boolean} Skip `value` and `offset` validation? **Default:** `false`
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `value` no `buf` no `offset` especificado com o formato endian especificado (`writeUInt32BE()` escreve em big endian, `writeUInt32LE()` escreve em little endian). `value` deve ser um inteiro válido não assinado de 32 bits. O comportamento é indefinido quando `value` é qualquer coisa diferente de um inteiro de 32 bits não assinado.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt32BE(0xfeedface, 0);

// Prints: <Buffer fe ed fa ce>
console.log(buf);

buf.writeUInt32LE(0xfeedface, 0);

// Prints: <Buffer ce fa ed fe>
console.log(buf);
```

### buf.writeUIntBE(value, offset, byteLength[, noAssert])

### buf.writeUIntLE(value, offset, byteLength[, noAssert])

<!-- YAML
added: v0.5.5
-->

* `value` {integer} Número a ser escrito para `buf`.
* `offset` {integer} Número de bytes para pular antes de começar a escrever. Must satisfy: `0 <= offset <= buf.length - byteLength`.
* `byteLength` {integer} Número de bytes para escrever. Must satisfy: `0 < byteLength <= 6`.
* `noAssert` {boolean} Skip `value`, `offset`, and `byteLength` validation? **Default:** `false`.
* Retorna: {integer} `offset` mais o número de bytes escritos.

Escreve `byteLength` bytes de `value` para `buf` no `offset` especificado. Suporta até 48 bits de precisão. O comportamento é indefinido quando `value` é qualquer coisa diferente de um inteiro não assinado.

Setting `noAssert` to `true` allows the encoded form of `value` to extend beyond the end of `buf`, but the resulting behavior is undefined.

Examples:

```js
const buf = Buffer.allocUnsafe(6);

buf.writeUIntBE(0x1234567890ab, 0, 6);

// Prints: <Buffer 12 34 56 78 90 ab>
console.log(buf);

buf.writeUIntLE(0x1234567890ab, 0, 6);

// Prints: <Buffer ab 90 78 56 34 12>
console.log(buf);
```

## buffer.INSPECT_MAX_BYTES

<!-- YAML
added: v0.5.4
-->

* {integer} **Default:** `50`

Retorna o número máximo de bytes que será retornado quando o `buf.inspect()` for chamado. Isto pode ser substituído por módulos de usuário. Veja [`util.inspect()`] para mais detalhes sobre o comportamento `buf.inspect()`.

Note that this is a property on the `buffer` module returned by `require('buffer')`, not on the `Buffer` global or a `Buffer` instance.

## buffer.kMaxLength

<!-- YAML
added: v3.0.0
-->

* {integer} O maior tamanho permitido para uma instância `Buffer` única.

Um apelido para [`buffer.constants.MAX_LENGTH`][]

Note that this is a property on the `buffer` module returned by `require('buffer')`, not on the `Buffer` global or a `Buffer` instance.

## buffer.transcode(source, fromEnc, toEnc)

<!-- YAML
added: v7.1.0
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/10236
    description: The `source` parameter can now be a `Uint8Array`.
-->

* `source` {Buffer|Uint8Array} Um `Buffer` ou uma instância `Uint8Array`.
* `fromEnc` {string} A codificação atual.
* `fromEnc` {string} A codificação futura.

Re-encodes the given `Buffer` or `Uint8Array` instance from one character encoding to another. Retorna uma nova instância `Buffer`.

Throws if the `fromEnc` or `toEnc` specify invalid character encodings or if conversion from `fromEnc` to `toEnc` is not permitted.

The transcoding process will use substitution characters if a given byte sequence cannot be adequately represented in the target encoding. Por exemplo:

```js
const buffer = require('buffer');

const newBuf = buffer.transcode(Buffer.from('€'), 'utf8', 'ascii');
console.log(newBuf.toString('ascii'));
// Saída: '?'
```

Because the Euro (`€`) sign is not representable in US-ASCII, it is replaced with `?` in the transcoded `Buffer`.

Note that this is a property on the `buffer` module returned by `require('buffer')`, not on the `Buffer` global or a `Buffer` instance.

## Class: SlowBuffer

<!-- YAML
deprecated: v6.0.0
-->

> Stability: 0 - Deprecated: Use [`Buffer.allocUnsafeSlow()`] instead.

Returns an un-pooled `Buffer`.

In order to avoid the garbage collection overhead of creating many individually allocated `Buffer` instances, by default allocations under 4KB are sliced from a single larger allocated object.

In the case where a developer may need to retain a small chunk of memory from a pool for an indeterminate amount of time, it may be appropriate to create an un-pooled `Buffer` instance using `SlowBuffer` then copy out the relevant bits.

Exemplo:

```js
// Need to keep around a few small chunks of memory
const store = [];

socket.on('readable', () => {
  const data = socket.read();

  // Allocate for retained data
  const sb = SlowBuffer(10);

  // Copy the data into the new allocation
  data.copy(sb, 0, 0, 10);

  store.push(sb);
});
```

Use of `SlowBuffer` should be used only as a last resort *after* a developer has observed undue memory retention in their applications.

### new SlowBuffer(size)

<!-- YAML
deprecated: v6.0.0
-->

> Stability: 0 - Deprecated: Use [`Buffer.allocUnsafeSlow()`] instead.

* `size` {integer} The desired length of the new `SlowBuffer`.

Aloca um novo `Buffer` de `size` bytes. If the `size` is larger than [`buffer.constants.MAX_LENGTH`] or smaller than 0, a [`RangeError`] will be thrown. A zero-length `Buffer` will be created if `size` is 0.

The underlying memory for `SlowBuffer` instances is *not initialized*. The contents of a newly created `SlowBuffer` are unknown and may contain sensitive data. Use [`buf.fill(0)`][`buf.fill()`] to initialize a `SlowBuffer` to zeroes.

Exemplo:

```js
const { SlowBuffer } = require('buffer');

const buf = new SlowBuffer(5);

// Prints: (contents may vary): <Buffer 78 e0 82 02 01>
console.log(buf);

buf.fill(0);

// Prints: <Buffer 00 00 00 00 00>
console.log(buf);
```

## Buffer Constants

<!-- YAML
added: 8.2.0
-->

Note that `buffer.constants` is a property on the `buffer` module returned by `require('buffer')`, not on the `Buffer` global or a `Buffer` instance.

### buffer.constants.MAX_LENGTH

<!-- YAML
added: 8.2.0
-->

* {integer} O maior tamanho permitido para uma instância `Buffer` única.

On 32-bit architectures, this value is `(2^30)-1` (~1GB). On 64-bit architectures, this value is `(2^31)-1` (~2GB).

This value is also available as [`buffer.kMaxLength`][].

### buffer.constants.MAX_STRING_LENGTH

<!-- YAML
added: 8.2.0
-->

* {integer} The largest length allowed for a single `string` instance.

Represents the largest `length` that a `string` primitive can have, counted in UTF-16 code units.

This value may depend on the JS engine that is being used.