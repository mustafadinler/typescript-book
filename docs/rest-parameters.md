### Rest Parametreleri
Rest parametreleri, belirsiz sayıda argümanı bir dizi olarak alan fonksiyonlar oluşturabilmenize olanak sağlar. Tanımlaması ise, fonksiyonun son argümanının `...argumentName` olarak ifade edilmesi ile gerçekleştirilir. Aşağıdaki örneği inceleyiniz.

```ts
function iTakeItAll(first, second, ...allOthers) {
    console.log(allOthers);
}
iTakeItAll('foo', 'bar'); // []
iTakeItAll('foo', 'bar', 'bas', 'qux'); // ['bas','qux']
```

Rest parametreleri herhangi bir fonksiyonda kullanılabilir  `function`/`()=>`/`class member`.