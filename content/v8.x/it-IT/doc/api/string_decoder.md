# String Decoder

<!--introduced_in=v0.10.0-->

> Stabilità: 2 - Stable

The `string_decoder` module provides an API for decoding `Buffer` objects into strings in a manner that preserves encoded multi-byte UTF-8 and UTF-16 characters. Ci si può accedere utilizzando:

```js
const { StringDecoder } = require('string_decoder');
```

L'esempio seguente mostra l'uso di base della `StringDecoder` class.

```js
const { StringDecoder } = require('string_decoder');
const decoder = new StringDecoder('utf8');

const cent = Buffer.from([0xC2, 0xA2]);
console.log(decoder.write(cent));

const euro = Buffer.from([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro));
```

When a `Buffer` instance is written to the `StringDecoder` instance, an internal buffer is used to ensure that the decoded string does not contain any incomplete multibyte characters. These are held in the buffer until the next call to `stringDecoder.write()` or until `stringDecoder.end()` is called.

In the following example, the three UTF-8 encoded bytes of the European Euro symbol (`€`) are written over three separate operations:

```js
const { StringDecoder } = require('string_decoder');
const decoder = new StringDecoder('utf8');

decoder.write(Buffer.from([0xE2]));
decoder.write(Buffer.from([0x82]));
console.log(decoder.end(Buffer.from([0xAC])));
```

## Class: new StringDecoder([encoding])

<!-- YAML
added: v0.1.99
-->

* `encoding`{string} La codifica del carattere che verrà usata da `StringDecoder`. **Default:** `'utf8'`.

Crea una nuova istanza `StringDecoder`.

### stringDecoder.end([buffer])

<!-- YAML
added: v0.9.3
-->

* `buffer` {Buffer} Un `Buffer` contenente i byte da decodificare.

Restituisce qualsiasi restante input memorizzato nel buffer interno come una stringa. Bytes representing incomplete UTF-8 and UTF-16 characters will be replaced with substitution characters appropriate for the character encoding.

If the `buffer` argument is provided, one final call to `stringDecoder.write()` is performed before returning the remaining input.

### stringDecoder.write(buffer)

<!-- YAML
added: v0.1.99
changes:

  - version: v8.0.0
    pr-url: https://github.com/nodejs/node/pull/9618
    description: Each invalid character is now replaced by a single replacement
                 character instead of one for each individual byte.
-->

* `buffer` {Buffer} Un `Buffer` contenente i byte da decodificare.

Returns a decoded string, ensuring that any incomplete multibyte characters at the end of the `Buffer` are omitted from the returned string and stored in an internal buffer for the next call to `stringDecoder.write()` or `stringDecoder.end()`.