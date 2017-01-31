### Spread (Yayma) Operatörü
Spread (Yayma) operatörünün ana hedefi bir dizinin elemanlarını *yaymaktır*. Bu operatörü açıklamak için örnekleri kullanacağız.

#### Apply
Yayma işleminin en yaygın kullanımlarından biri, dizinin elemanlarını bir fonksiyonun argümanları olarak kullanmaktır. Önceki sürümlerde bu işlemi yapmak için `Function.prototype.apply` ifadesini kullanmanız gerekmekteydi. 

```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo.apply(null, args);
```

Şimdi ise bu işlemi yapmak için gereken tek şey, aşağıdaki örnekte de görüleceği üzere, fonksiyon argümanlarınızın önüne `...` eklemektir.

```ts
function foo(x, y, z) { }
var args = [0, 1, 2];
foo(...args);
```

Yukarıdaki örnekte `args` dizisini fonksiyon argümanlarına dönüşecek şekilde yayıyoruz.

#### Parçalama (Sökme)
Bu konunun bir örneğini *destructuring* bölümünde görmüştük.

```ts
var [x, y, ...remaining] = [1, 2, 3, 4];
console.log(x, y, remaining); // 1, 2, [3,4]
```
Bu örnekteki amaç, diziyi parçalarken dizide kalan eşleşmemiş elemanları kolay bir şekilde yakalayarak bir değişkene atamaktır.  

#### Dizi Atama (Birleştirme)
Yayma operatörü, bir dizinin *yayılmış (genişletilmiş) versiyonunu* kolaylıkla başka bir dizinin içerisine yerleştirmenize olanak sağlar. Aşağıda bunun bir örneğini görebilirsiniz:

```ts
var list = [1, 2];
list = [...list, 3, 4];
console.log(list); // [1,2,3,4]
```

#### Özet
`apply` fonksiyonu JavaScript ile çalışırken kaçınılmaz olarak kullanacağınız bir ifadedir, bu sebeple `apply` fonksiyonunun ilk argümanı olan `this` yerine uygunsuz ve neredeyse çirkin denilebilecek bir sözdizimine yol açan `null` kullanmak yerine daha düzgün bir sözdizimi getiren *spread (yayma) operatörü* kullanmayı tercih edebilirsiniz. 
Bununla birlikte  *spread (yayma) operatörü*,  kısmı diziler üzerinden yapılan parçalama ve birleştirme işlemleri için çok daha zarif bir sözdizimi sunmaktadır.

[](https://github.com/Microsoft/TypeScript/pull/1931)
