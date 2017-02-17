### for...of
Dizi öğelerinin yinelenmesi için `for...in` kullanılması, Javascript'e yeni başlayan geliştiricilerin karşılaştıkları yaygın bir hatadır. Dizi elemanları yerine verilen nesnenin *anahtarlarını*(*keys*) yineler. Aşağıdaki örnekte gösterilmiştir. `9,2,5` beklemenize rağmen, çıkan sonuç `0,1,2` dizinleridir:

```ts
var someArray = [9, 2, 5];
for (var item in someArray) {
    console.log(item); // 0,1,2
}
```

`for...of`'un TypeScript'te (ve ES6'da) olma sebeplerinden biri, işte budur. Örneğin, sıradaki dizi doğru bir biçimde yinelenmekte ve elemanları beklenildiği gibi çıkmaktadır:

```ts
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item); // 9,2,5
}
```

TypeScript, karakter dizimindeki karakterleri yinelemek için de `for...of` kullanmanıza imkan sağlar:

```ts
var selam = "naber?";
for (var char of selam) {
    console.log(char); 
}
// 'n'
// 'a'
// 'b'
// 'e'
// 'r'
// '?'
```

#### JS Oluşturulması
TypeScript, ES6 öncesi için `for (var i = 0; i < list.length; i++)` benzeri standart bir döngü oluşturur. Önceki örneğimizin Javascript'te oluşturulmuş hali, şu şekildedir:
```ts
var someArray = [9, 2, 5];
for (var item of someArray) {
    console.log(item);
}

// şuna dönüşür //

for (var _i = 0; _i < someArray.length; _i++) {
    var item = someArray[_i];
    console.log(item);
}
```
Gördüğünüz gibi `for...of` kullandığınızda *maksadınız* daha açıktır. Ayrıca yazdığınız kod miktarı (ve kullandığınız değişken isimleri) azalır.

#### Kısıtlamalar
Eğer ES6 veya üstünü hedeflemiyorsanız, oluşturulan kod, nesnede `length` özelliği olduğunu varsayar ve nesneye dizin kullanarak erişir, örneğin; `obj[2]`. Bu yüzden eski JS motorları sadece `string` ve `array` destekler.

Eğer dizi veya karakter dizimi kullanmazsınız, TypeScript bunu görür ve *"dizi veya karakter dizimi tipinde değil"* hatası verir;
```ts
let articleParagraphs = document.querySelectorAll("article > p");
//Hata: Nodelist is not an array type or a string type
for (let paragraph of articleParagraphs) {
    paragraph.classList.add("read");
}
```
`for...of`'u sadece dizi veya karakter dizimi olduğunu *bildiğiniz* şeylerde kullanın. Bu kısıtlamanın TypeScript'in ileriki sürümlerinde kaldırılabileceğini de göz önünde bulundurunuz.

#### Özet
Dizilerde yinelemeyi ne kadar kullandığınızı bilseniz şaşırırsınız. Bir dahaki denemenizde, `for...of`'a bir şans verin. Belki de kodunuzu gözden geçiren bir sonraki kişiyi mutlu edeceksiniz. 