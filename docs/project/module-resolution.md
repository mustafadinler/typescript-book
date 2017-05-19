# TypeScript Module Resolution

TypeScript's module resolution tries to model and support the real world modules systems / loaders there (commonjs/nodejs, amd/requirejs, ES6/systemjs etc.). En basit arama şekli relatif dosya yoluna bakmaktır. Bundan sonra ise işler *çeşitli modül yükleyicileri tarafından yapılan sihirli modül yüklemelerinin doğası gereği* birazcık karışmaya başlar.

## Dosya Uzantıları (File Extensions)

Modülleri `foo` veya `./foo` gibi içe aktarırsınız. For any file path lookup TypeScript automatically checks for a `.ts` or `.d.ts` or `.tsx` or `.js` (optionally) or `.jsx` (optionally) file in the right order depending upon context. You should **not** provide a file extension with the module name (no `foo.ts`, just `foo`).

## Relatif Dosya Modülü (Relative File Module)

Relatif yol ile içe aktarmaya örnek olarak:

```ts
import foo = require('./foo');
```

Tells the TypeScript compiler to look for a TypeScript file at the relative location e.g. `./foo.ts` with respect to the current file. There is no further magic to this kind of import. Of course it can be a longer path e.g. `./foo/bar/bas` or `../../../foo/bar/bas` just like any other *relative paths* you are used to on disk.

## Named Module

Aşağıdaki ifade:

```ts
import foo = require('foo');
```

Typescript derleyicisine aşağıdaki sıra ile harici bir modül için arama yapmasını söyler:

* A named [module declaration](#module-declaration) from a file already in the compilation context.
* If still not resolved and you are compiling with `--module commonjs`  or have set `--moduleResolution node` then its looked up using the [*node modules*](#node-modules) resolution algorithm.
* If still not resolved and you provided `baseUrl` (and optionally `paths`) then the [*path substitutions*](#path-substitutions) resolution algorithm kicks in.

Note that `"foo"` can be a longer path string e.g. `"foo/bar/bas"`. The key here is that *it does not start with `./` or `../`*.

## Modül Deklarasyonu (Module Declaration)

Modül deklarasyonu şöyle gözükür:

```ts
declare module "foo" {

    /// Bazı değişken deklarasyonları

    export var bar:number; /*örnek*/
}
```

Bu `"foo"` modülünü, *içe aktarılamaz* yapar.

## Node Modülleri (Node Modules)
The node module resolution is actually pretty much the same one used by NodeJS / NPM ([official nodejs docs](https://nodejs.org/api/modules.html#modules_all_together)). Here is a simple mental model I have:

* module `foo/bar` will resolve to some file : `node_modules/foo` (the module) + `foo/bar`

## Path Substitutions

TODO.

[//Comment1]:https://github.com/Microsoft/TypeScript/issues/2338
[//Comment2]:https://github.com/Microsoft/TypeScript/issues/5039
[//Comment3ExampleRedirectOfPackageJson]:https://github.com/Microsoft/TypeScript/issues/8528#issuecomment-219172026
[//Coment4ModuleResolutionInHandbook]:https://github.com/Microsoft/TypeScript-Handbook/blob/release-2.0/pages/Module%20Resolution.md#base-url
