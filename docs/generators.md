## Oluşturucular

> NOT: TypeScript'te oluşturucular henüz olgunlaşmamıştır. Yine de olgunlaştırma çalışmaları olduğundan bu konuyu ekledik.  

TypeScript'te `function *` deyimi kullanılarak bir *oluşturucu fonksiyon* tanımlanabilir. Oluşturucu fonksiyonun geridönüşü bir *oluşturucu nesne*dir ve [yineleyici][iterator] arayüzünü uygulamaktadır. Bu bağlamda `next`, `return` ve `throw` çağırımlarına uygundur. Oluşturucu fonksiyonlar iki temel amaca sahiptir.

### Geç Yineleyiciler

Aşağıda **sonsuz** sayıda tamsayı dönen fonksiyon örneğindeki gibi, oluşturucu fonksiyonlar geç yineleyicileri tanımlamak için kullanılabilir.  

```ts
function* infiniteSequence() {
    var i = 0;
    while(true) {
        yield i++;
    }
}

var iterator = infiniteSequence();
while (true) {
    console.log(iterator.next()); // { value: xxxx, done: false } - işlem sonsuza kadar devam eder
}
```

ancak yineleyicinin bir sonu olduğu koşulda aşağıda gözlemleyebileceğiniz şekilde `{done:true}` sonucu alınabilir:

```ts
function* idMaker(){
  let index = 0;
  while(index < 3)
    yield index++;
}

let gen = idMaker();

console.log(gen.next()); // { value: 0, done: false }
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { done: true }
```

### Dışarıdan Kontrol Edilen Yürütmeler
Yürütülen bir fonksiyonu duraklatmak ve kalan yürütme işinin kontrolünü o fonksiyonu çağırana iletmek oluşturucuların temel işlevidir. 
Bu, oluşturucuların keyifli ve heyecan verici yanıdır.

Oluşturucu fonksiyonlar çağırıldıkları anda yürütülmezler, yalnızca nesne oluştururlar.
Aşağıdaki örneği ve örnek çağırımı inceleyelim:

```ts
function* generator(){
    console.log('Execution started');
    yield 0;
    console.log('Execution resumed');
    yield 1;
    console.log('Execution resumed');
}

var iterator = generator(); // Oluşturucu nesnesi
console.log('Starting iteration'); // Oluşturucu fonksiyonun içeriğinden önce bu satır çalışacaktır.
console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

Örneğin çalıştırılması durumunda aşağıdaki çıktı gözlemlenebilir:

```
$ node outside.js
Starting iteration
Execution started
{ value: 0, done: false }
Execution resumed
{ value: 1, done: false }
Execution resumed
{ value: undefined, done: true }
```

* Oluşturucu fonksiyon (`generator()`) ancak oluşturucu nesnesindeki `next` fonksiyonunun çağırımıyla yürütülmeye başlar.  
* Oluşturucu fonksiyon `yield` deyimi ile *duraklatılır* *(pause)*.
* Oluşturucu fonksiyon `next` çağırımı yapıldığında yürütülmeye *devam eder* *(resume)*.

> Oluşturucu fonksiyonun yürütülmesi, oluşturucu nesne tarafından kontrol edilebilir durumdadır.

Oluşturucuları çoğu zaman *tek yönlü* olarak, yani sadece yineleyiciden geri döndürülen değerleri için kullanırız.
Ancak JavaScript'in gerçeklen güçlü bu ozelliği aslında *iki yönlü* olarak kullanıma olanak verir.

* `iterator.next(valueToInject)` kullanılarak `yield` ile döndürülen değer kontrol edilebilir veya değiştirilebilir.
* `iterator.throw(error)` kullanılarak `yield` işleyişinde istisna oluşturulabilir.

`iterator.next(valueToInject)` örneği:

```ts
function* generator() {
    var bar = yield 'foo';
    console.log(bar); // bar!
}

const iterator = generator();
// yield'dan dönen ilk değeri alana kadar yürütmeyi başlat
const foo = iterator.next();
console.log(foo.value); // foo
// bar değerini işleme alarak yürütmeye devam et
const nextThing = iterator.next('bar');
```

`iterator.throw(error)` örneği:

```ts
function* generator() {
    try {
        yield 'foo';
    }
    catch(err) {
        console.log(err.message); // bar!
    }
}

var iterator = generator();
// yield'dan dönen ilk değeri alana kadar yürütmeyi başlat
var foo = iterator.next();
console.log(foo.value); // foo
// 'bar' mesajıyla bir istisna oluşturarak yürütmeye devam et
var nextThing = iterator.throw(new Error('bar'));
```

Özetleyecek olursak:
* `yield` deyimi bir oluşturucu fonksiyonun işleyişini duraksatarak, yürütme kontrolünü dış yapıya iletir.
* oluşturucu fonksiyonlardaki değerler dışarıdan manipüle edilebilir. 
* oluşturucu fonksiyonların işleyişleri istisnalara ve hatalara göre şekillendirilebilir.

Bu konuyla ilgili olarak bir sonraki bölüm olan [**async/await**][async-await] konusuna bir göz atın.

[iterator]:./iterators.md
[async-await]:./async-await.md
