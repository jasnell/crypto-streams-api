# crypto-streams-api

*This is an initial draft* intended to spark a conversation about what an actual Crypto Streams API should be. 

## What is the goal here?

The Web Crypto API does not support streaming. That's a problem I'd like to fix.

Specifically, it would be great if we could introduce new APIs for generating a hash digest from a stream, generating signature and verification from a stream, and encrypting/decrypting from a stream, with all that done in a way that is compatible with and familiar to both the Web Crypto and WHATWG Streams APIs.

## A Conversation Starter

```webidl
interface DigestStream : WritableStream {
  constructor(AlgorithmIdentifier);
  
  readonly attribute Promise<ArrayBuffer> digest;
};

interface SignStream : WritableStream {
  constructor(AlgorithmIdentifier, CryptoKey);
  
  readonly attribute Promise<ArrayBuffer> sign;
};

interface VerifyStream : WritableStream {
  constructor(AlgorithmIdentifier, CryptoKey, BufferSource);
  
  readonly attribute Promise<any> verify;
};

interface EncryptStream : TransformStream {
  constructor(AlgorithmIdentifier, CryptoKey, StreamCipherParams);
};

interface DecryptStream : TransformStream {
  constructor(AlgorithmIdentifier, CryptoKey, StreamCipherParams);
};

callback BlockCallback = Promise<undefined>(StreamCipherController controller);

dictionary StreamCipherParams {
  unsigned long blockSize;
  
  // The block callback is called on the completion of each block if the stream
  // cipher supports it, giving user code the opportunity to update the cipher
  // options for each subsequent block, if appropriate.
  BlockCallback block;  
};

interface StreamCipherController {
  void update(AlgorithmIdentifier, CryptoKey);
  void error(any);
  void terminate();
};
```

The intent here is for each of the new APIs to closely align with both Web Crypto and Web Streams.

## Examples

### DigestStream

```js
const ds = new DigestStream('SHA-256');
const writer = ds.getWriter();
const enc = new TextEncoder();
enc.write('hello');
enc.write('world');
enc.close();

console.log(await ds.digest);
```

### SignStream

```js
const key = await crypto.subtle.importKey(/* ... */);
const ds = new SignStream({ name: 'RSASSA-PKCS1-v1_5' }, key);
const writer = ds.getWriter();
const enc = new TextEncoder();
enc.write('hello');
enc.write('world');
enc.close();

console.log(await ds.sign);
```

### VerifyStream

```js
const key = await crypto.subtle.importKey(/* ... */);
const ds = new VerifyStream({ name: 'RSASSA-PKCS1-v1_5' }, key, signature);
const writer = ds.getWriter();
const enc = new TextEncoder();
enc.write('hello');
enc.write('world');
enc.close();

console.log(await ds.verify);
```

### EncryptStream

```js
const key = await crypto.subtle.importKey(/* ... */);
const es = new EncryptStream({ name: '...' }, key);
const readable = getReadableSomehow().pipeThrough(es);
// readable outputs encrypted bytes
```

### DecryptStream

```js
const key = await crypto.subtle.importKey(/* ... */);
const es = new DecryptStream({ name: '...' }, key);
const readable = getReadableSomehow().pipeThrough(es);
// readable outputs plaintext bytes
```

## Key Questions and Open Issues

* Should the new objects be globals or hang off the existing `crypto` or `crypto.subtle` objects (e.g. should it be `new DigestStream()` or `new crypto.DigestStream()`)
* We'll need to define the set of streaming ciphers supported.

