# JavaScript'in kötü tarafları

İşte JavaScript'in bilmeniz gereken bazı kötü (anlaşılmaz) tarafları.

> Not: TypeScript, JavaScript'in süpersetidir. Aslında derleyiciler/IDE'ler tarafından kullanılan dokümantasyonlu halidir ;)

## Null ve Undefined

Gerçek şu ki ikisiyle de uğraşmanız gerekecektir. İkisi için de `==` ile kontrol edelim.

```ts
/// bar aşagıdakilerden biri iken `foo.bar == undefined` yaptığınızı düşünün:
console.log(undefined == undefined); // true
console.log(null == undefined); // true
console.log(0 == undefined); // false
console.log('' == undefined); // false
console.log(false == undefined); // false
```
Tavsiyem `undefined` veya `null`, ikisi için de `== null` ile kontrol etmelisiniz. Genellikle bu ikisi arasında bir ayrım yapmak istemezsiniz.

## undefined

Hatırlarsanız `== null` kullanmanız gerektiğini söylemiştim. Tabii ki kullanacaksınız (çünkü ben öyle söyledim ^). Kök seviyesindeki şeyler için kullanmayın. Katı modda (strict mode) eğer `foo` kullanırsanız ve `foo` tanımlanmamış (undefined) ise, `ReferenceError` **hatası** alırsınız ve tüm çağrı kümeniz bozulur.

> Katı mod (strict mode) kullanmalısınız ... ve aslında TS derleyicisi eğer modül kullanırsanız sizin için bunu ekleyecektir ... daha fazlası kitabın ilerleyen bölümlerinde var yani şu anda açık şekilde anlamış olmanız gerekmez :)

Yani bir değişkenin *global* düzeyde tanımlanıp tanımlanmadığını kontrol etmek için `typeof` kullanılır:

```ts
if (typeof someglobal !== 'undefined') {
  // someglobal şu anda kullanım için güvenlidir
  console.log(someglobal);
}
```

## this

Bir fonksiyon içindeki `this` anahtar sözcüğüne herhangi bir erişim, aslında fonksiyonun nasıl çağrıldığına göre kontrol edilir. Genellikle `çağrı bağlamı (calling context)` olarak bahsedilir.

Aşağıdaki örnek bunu açıklamaktadır:

```ts
function foo() {
  console.log(this);
}

foo(); // global olarak çıktı verir, örneğin tarayıcılardaki `window` 
let bar = {
  foo
}
bar.foo(); // `bar` üzerinde çağrıldığında `foo` olarak `bar` çıktısı verir
```
Bu nedenle `this` kullanımınızda dikkatli olun . Eğer çağırdığınız bağlamda gelen sınıfın icindeki `this`'ten kopmak isterseniz ok fonksiyonlarını kullanın, [daha fazlası][ok].

[ok]:../arrow-functions.md

## Sıradaki

İşte bu. Bunlar JavaScript'in bazı yanlış anlaşılan parçalarıdır ve hala bu dilde yeni olan geliştiriciler için çeşitli hatalarla sonuçlanır 🌹.