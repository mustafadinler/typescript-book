* [Ok Fonksiyonları](#arrow-functions)
* [Tüyo: Ok Fonksiyonu Gereksinimi](#tip-arrow-function-need)
* [Tüyo: Tehlikeli Ok Fonksiyonu Kullanımı](#tip-arrow-function-danger)
* [Tüyo: `this` kullanan kütüphaneler](#tip-arrow-functions-with-libraries-that-use-this)
* [Tüyo: Arrow Function Kalıtımı](#tip-arrow-functions-and-inheritance)

### Ok Fonksiyonları

*Kalın ok* (çünkü `->` ince ok, `=>` ise kalın oktur) veya *lambda fonksiyonu* (diğer dillerde geçtiği şekliyle) olarak isimlendirilir. `()=>something`, kalın ok fonksiyonunun çokça kullanılan bir özelliğidir. Kalın ok fonksiyonunun kullanılmasının nedenleri:
1. `function` yazmanıza gerek olmaz.
2. Anlam olarak `this` sözcüğünü içerir.
3. Anlam olarak `arguments` sözcüğünü içerir.

İşlevsel (fonksiyonel) olduğunu iddia eden bir dil olarak JavaScript'te fazlasıyla `function` yazma eğiliminde olursunuz. Kalın ok size kolay bir şekilde fonksiyon yaratmanızı sağlar.
```ts
var inc = (x)=>x+1;
```
`this` geleneksek olarak Javascript'te sıkıntılı bir konudur. Bilge bir adam der ki, "`this`, çok kolay bir biçimde anlamını yitirdiği için Javascript'ten nefret ettim." Kalın ok bu durumu `this`'in anlamını etrafındaki içerikle yansıtarak düzeltti. Şu saf Javascript sınıfına bakarsak:

```ts
function Person(age) {
    this.age = age;
    this.growOld = function() {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 1, ama 2 olmalıydı
```
Eğer bu kodu tarayıcıda koşturursanız, fonksiyon içindeki `this`, `window`'u işaret eder çünkü `growOld` fonksiyonunu `window` çalıştıracaktır. Bunu düzeltmek için ok fonksiyonu kullanın:
```ts
function Person(age) {
    this.age = age;
    this.growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
Bunun işe yaramasının sebebi, ok fonksiyonunun `this` referansını fonksiyon gövdesinin dışından almasıdır. Buna eşdeğer Javascript kodu (TypeScript kullanmadan da yazabilirsiniz) ise aşağıdaki gibidir:
```ts
function Person(age) {
    this.age = age;
    var _this = this;  // this'i referans alır
    this.growOld = function() {
        _this.age++;   // referans alınan this kullanılır
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```
Ayrıca artık TypeScript kullandığınız için, daha güzel bir sözdizimi ile ok fonksiyonlarını sınıflar ile kullanabilirsiniz: 
```ts
class Person {
    constructor(public age:number) {}
    growOld = () => {
        this.age++;
    }
}
var person = new Person(1);
setTimeout(person.growOld,1000);

setTimeout(function() { console.log(person.age); },2000); // 2
```

#### Tüyo: Ok Fonksiyonu Gereksinimi
Kısa ve öz sözdiziminin dışında, sadece fonksiyonunuz başkası tarafından çağrılacaksa, kalın ok kullanmanız *gereklidir*. Örneğin:
```ts
var growOld = person.growOld;
// Ve sonra başkası çağırdığında:
growOld();
```
Eğer fonksiyonu kendiniz çağıracaksanız, örneğin;
```ts
person.growOld();
```
İşte o zaman `this` doğru bir içeriği çağırmış olur (`person` örneğindeki gibi).

#### Tüyo: Tehlikeli Ok Fonksiyonu Kullanımı

Aslında `this`'i *ilgili içeriği çağırmak için* istiyorsanız, *ok fonksiyonu kullanmamalısınız*. jquery, underscore, mocha gibi geri çağırımlar (callback) kullanan kütüphaneler bu duruma örnektir. Eğer dökümanında fonksiyonlarda `this` kullanımından bahsediyorsa, o zaman kalın ok yerine muhtemelen `function` kullanmanız gerekiyor. Aynı şekilde eğer `arguments` kullanmayı düşünüyorsanız da, kalın ok fonksiyonu kullanmamalısınız.

#### Tüyo: Ok fonksiyonları ve `this` kullanan kütüphaneler
Birçok kütüphane bunu yapmaktadır. Örneğin, `jQuery` yineleyicileri (iterable) (bir örneği http://api.jquery.com/jquery.each/) o an yinelenmekte olan nesneye `this`'i iletecektir. Bu durumda, `this`'i ve çevreleyen içeriğini ileten bir kütüphaneye erişmek istiyorsanız, benzeri bir geçici değişken kullanın: 

```ts
let _self = this;
something.each(function() {
    console.log(_self); // sözcük anlamı olarak kapsamın değeri
    console.log(this); // kütüphanenin ilettiği değer
});
```

#### Tip: Ok fonksiyonları ve kalıtım

Ok fonksiyonu ile yazılıp `this` ile devam eden, bir sınıfa ait bir metodunuz var. Sadece tek bir `this` olduğundan,  bu tip fonksiyonlar `super`'i (`super` sadece prototip üyelerinde çalışır) çağıramazlar. Alt sınıf metodun üzerinden geçmeden önce, metodun kopyasını yaratarak bunun üstesinden gelebilirsiniz.

```ts
class Adder {
    constructor(public a: number) {}
    // Bu fonksiyon şu an çağrılabilmeye uygun
    add = (b: string): string => {
        return this.a + b;
    }
}

class ExtendedAdder extends Adder {
    // Kendi metodundan önce üst sınıfınkinin bir kopyasını yarat
    private superAdd = this.add;
    // Şimdi onun üzerinden geçen kendi metodunu yarat
    add = (b: string): string => {
        return this.superAdd(b);
    }
}
```
