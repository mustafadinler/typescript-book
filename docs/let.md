### let

JavaScript'deki `var` değişkenleri *fonksiyon faaliyet alan*'lıdır. Bu, değişkenlerin *blok faaliyet alanlı* olduğu diğer birçok dilden farklidir (C# / Java vb.). Eğer JavaScript'e *blok faaliyet alanlı* düşünce yapısı getirirseniz, aşağıdakilerin `123` çıktısı vermesini beklersiniz, bunun yerine  `456` yazdıracaktır:

```ts
var foo = 123;
if (true) {
    var foo = 456;
}
console.log(foo); // 456
```
Bunun nedeni ise `{` yeni bir *değişken faaliyet alanı* yaratmaz. foo değişkeni, if *bloğunun* içinde olduğu gibi if bloğunun dışında da aynıdır. Bu, JavaScript programlamada yaygın bir hata kaynağıdır. Bu nedenle TypeScript (ve ES6) size değişkenleri doğru *blok faaliyet alan*'ları ile tanımlayabilmeniz için `let` anahtar sözcüğünü sunar. Yani, `var` yerine `let` kullanırsanız, faaliyet alanının dışında tanımladıklarınızdan kopuk eşsiz bir eleman elde edersiniz. Aynı örneğin `let` ile gösterimi :

```ts
let foo = 123;
if (true) {
    let foo = 456;
}
console.log(foo); // 123
```

`let`'in sizi hatalardan kurtaracağı bir başka yer ise döngülerdir.
```ts
var index = 0;
var array = [1, 2, 3];
for (let index = 0; index < array.length; index++) {
    console.log(array[index]);
}
console.log(index); // 0
```
Samimiyetle, yeni ve mevcut çok dilli geliştiriciler için `let` kullanımını, daha az sürprizlerle karşılaşacaklarından uygun buluyoruz.

#### Fonksiyonlar yeni faaliyet alanı oluşturur
Bahsettiğimizden beri, JavaScript'te fonksiyonların yeni bir değişken faaliyet alanı oluşturduğunu göstermek istiyoruz. Aşağıdaki gösterimi inceleyelim:

```ts
var foo = 123;
function test() {
    var foo = 456;
}
test();
console.log(foo); // 123
```
Bu beklediğiniz gibi davranıyor. Aksi takdirde JavaScript'de kod yazmak çok zor olurdu.

#### Üretilmiş JS
TypeScript tarafından üretilen JS basitçe, let değişkenini çevreleyen faaliyet alanında benzer bir isim zaten varsa yeniden adlandırılmasıdır. Örneğin aşağıdaki `var` değişkeninin `let` ile basit bir yer değiştirmesi ile üretilir:

```ts
if (true) {
    let foo = 123;
}

// şu hale gelir //

if (true) {
    var foo = 123;
}
```
Ancak değişken ismi çevreleyen faaliyet alanı tarafından zaten alındıysa, yeni değişken ismi gösterildiği gibi üretilir (bakınız `_foo`):

```ts
var foo = '123';
if (true) {
    let foo = 123;
}

// şu hale gelir //

var foo = '123';
if (true) {
    var _foo = 123; // yeniden adlandırılır
}
```

#### kapsanımlar içindeki let
Bir JavaScript geliştirici için alışılageldik bir programlama sorusu, bu basit dosya ne çıktı üretir:

```ts
var funcs = [];
// birkaç fonksiyon oluştur
for (var i = 0; i < 3; i++) {
    funcs.push(function() {
        console.log(i);
    })
}
// ve onları çağır
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```
Birileri bunun `0,1,2` olmasını beklerdi. Şaşırtıcı bir şekilde her üç işlev için `3` olacak. Bunun nedeni, her üç işlevin de `i` değişkenini dış faaliyet alanında kullanıyor olması ve o anda bunları çalıştırdığımızda (ikinci döngü içinde) `i`'nin değerinin `3` olması (bu, ilk döngü için sonlandırma koşuludur).

Doğrusu, o döngü tekrarlamasına özgü her döngüde yeni bir değişken oluşturmak olacaktır. Daha önce öğrendiğimiz gibi, yeni bir fonksiyon oluşturup onu anında çalıştırarak yeni bir değişken faaliyet alanı oluşturabiliriz, (örneğin sınıflardaki IIFE kalıbı `(function() { /* body */ })();`) aşağıda gösterildiği gibi:

```ts
var funcs = [];
// birkaç fonksiyon oluştur
for (var i = 0; i < 3; i++) {
    (function() {
        var local = i;
        funcs.push(function() {
            console.log(local);
        })
    })();
}
// ve onları çağır
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```
Burada fonksiyonlar, *yerel* değişkeni (uygun isimlendirmesi `yerel`) kapatıyor (dolayısıyla `kapsanım` denir) ve bunu, döngü değişkeni `i` yerine kullanıyor.

> Kapsanımların performans etkisine sahip olduğunu unutmayın (çevreleyen durumu tutmak zorundalar).

Döngü içerisindeki ES6 `let` anahtar sözcüğü, önceki örnekteki gibi aynı davranışa sahip olurdu:

```ts
var funcs = [];
// birkaç fonksiyon oluştur
for (let i = 0; i < 3; i++) { // let kullanımını unutmayın
    funcs.push(function() {
        console.log(i);
    })
}
// ve onları çağır
for (var j = 0; j < 3; j++) {
    funcs[j]();
}
```

`var` yerine `let` kullanılması, her döngü tekrarlanmasında eşsiz bir `i` değişkeni yaratır.

#### Özet
`let` kodun büyük çoğunluğuna hakim olmak için son derece kullanışlıdır. Kodunuzun okunabilirliğini büyük ölçüde artırabilir ve programlama hatalarının olasılığını azaltır.



[](https://github.com/olov/defs/blob/master/loop-closures.md)
