#### IIFE Nedir Ne Değildir?
Sınıf için üretilen js şöyle olabilirdi:
```ts
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.add = function (point) {
    return new Point(this.x + point.x, this.y + point.y);
};
```

Sebebi  Anlık-Çağrılan Fonksiyon İfadesi (Immediately-Invoked Function Expression) (IIFE) ile sarmalanmıştır, yani:

```ts
(function () {

    // BODY

    return Point;
})();
```

kalıtım ile yapar. TypeScript'in üst sınıfını `_super` değişkeni olarak yakalamasına olanak tanır örnegin :

```ts
var Point3D = (function (_super) {
    __extends(Point3D, _super);
    function Point3D(x, y, z) {
        _super.call(this, x, y);
        this.z = z;
    }
    Point3D.prototype.add = function (point) {
        var point2D = _super.prototype.add.call(this, point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    };
    return Point3D;
})(Point);
```

Dikkat edin IIFE, TypeScript'in kolayca üst sınıf `Point` içerisindeki `_super` değişkenini yakalamasına izin verir ve sınıfın ana bloğunda sürekli olarak kullanılır.

### `__extends`
Şunu farkedeceksiniz ki bir sınıftan kalıtım yaptığınız anda TypeScript ayrıca aşağıdaki fonksiyonu da oluşturur:
```ts
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```
Burada `d` türetilmiş sınıfı ve `b` üst sınıfı ifade eder. Bu fonksiyon iki şey yapar:
1. `for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];` yani üst sınıfının statik elemanlarını alt sınıfının üzerine kopyalar 
1. `d.prototype.__proto__ = b.prototype` yani etkin biçimde, alt sınıfının fonksiyonlarının prototiplerini isteğe bağlı olarak arama elemanlarına üst sınıfının `proto`'ları üzerinde atama yapar

İnsanlar nadiren 1'i anlamakta zorlanırlar, ancak birçok kişi 2 ile mücadele verir. Bu yüzden sırası ile açıklamaları.

#### `d.prototype.__proto__ = b.prototype`

Birçok insana bu konuyu anlattıktan sonra aşağıdaki açıklamayı en basit buluyorum. Öncelikle `__extends`'deki kodun nasıl basit `d.prototype.__proto__ = b.prototype`'a eşit olduğunu açıklayacağız, ve sonrasında kendi içindeki bu satırın neden değerli olduğunu. Bunu tamamen anlayabilmeniz için şunları bilmeniz gerekir:

1. `__proto__`
1. `prototype`
1. çağrılan fonksiyon içerisinde `new`'ın `this` üzerindeki etkisi
1. `new`'ın `prototype` ve `__proto__` üzerindeki etkisi

JavaScript'teki tüm nesneler `__proto__` elemanını içerir. Bu eleman eski tarayıcılar tarafından sıklıkla erişilemez durumdadır (bazen dökümantasyon bu sihirli özelliği `[[prototype]]` olarak ifade eder). Bunun tek bir görevi vardır: Eğer arama sırasında nesne üzerinde özellik bulunamazsa (örneğin `obj.property`) `obj.__proto__.property`'e bakacaktır. Eğer hala bulunamadıysa ozaman `obj.__proto__.__proto__.property`'a ta ki ikiside: *it is found* veya *the latest `.__proto__` itself is null* olana kadar. Bu JavaScript'in *prototipsel kalıtım(prototypal inheritance)* işlevselliğine neden destek dediğini açıklar. Bu aşağıdaki örnekte, chrome konsolunda veya nodejs'te çalıştırılacak şekilde gösterilmiştir:

```ts
var foo = {}

//  foo.__proto__'da olduğu gibi foo üzerinde hazırlanışı
foo.bar = 123;
foo.__proto__.bar = 456;

console.log(foo.bar); // 123
delete foo.bar; // nesneden çıkar
console.log(foo.bar); // 456
delete foo.__proto__.bar; // foo.__proto__'dan çıkar
console.log(foo.bar); // undefined
```

`__proto__`'yu anlıyorsun harika. Başka bir yararlı bilgi ise, JavaScript içerisindeki tüm `fonksiyon`'lar `prototype` diye adlandırılan özelliğe sahiptir ve fonksiyonu geri işaret eden `constructor` elemanına sahiptir. Aşağıda gösterilmiştir:

```ts
function Foo() { }
console.log(Foo.prototype); // {} yani zaten var ve tanımlanmamış
console.log(Foo.prototype.constructor === Foo); // `constructor` adında fonksiyonu geri işaret eden elemana sahip
```

Şimdi hadi *çağrılan fonksiyon içerisinde `new`'ın `this` üzerindeki etkisi*'ne bakalım. Aslında çağrılan fonksiyon içerisindeki `this`, fonksiyondan geri dönecek yeni yaratılmış nesneyi işaret edecektir. Eğer fonksiyon içerisindeki `this` üzerinde özelliği değiştirirseniz bunu görmek çok basittir:

```ts
function Foo() {
    this.bar = 123;
}

// new operatörü ile çağrımı
var newFoo = new Foo();
console.log(newFoo.bar); // 123
```

Şimdi öğrenmeniz gereken diğer son şey ise,fonksiyon üzerinde `new`'ı çağırmak, fonksiyon çağrısından dönen yeni yaratılmış nesnenin `__proto__`'su içindeki `prototype`'ı kopyalar. İşte çalıştırıp tamamen anlayabilmeniz için gereken kod bu:

```ts
function Foo() { }

var foo = new Foo();

console.log(foo.__proto__ === Foo.prototype); // True!
```

İşte bu. Şimdi `__extends`'e göz atın. Bu satırları numaralandırma özgürlüğünü kendimde buldum:

```ts
1  function __() { this.constructor = d; }
2   __.prototype = b.prototype;
3   d.prototype = new __();
```

Bu fonksiyonu satır 3 üzerindeki `d.prototype = new __()` tersten okumanın etkili anlamı `d.prototype = {__proto__ : __.prototype}` (`new`'ın `prototype` ve `__proto__` üzerindeki etkisi yüzünden), önceki satırdaki (yani satır 2 `__.prototype = b.prototype;`) ile birleştirme sonucunda `d.prototype = {__proto__ : b.prototype}` alırsınız.

Ama durun, biz `d.prototype.__proto__` istedik yani sadece prototip değişsin ve eski `d.prototype.constructor` sürsün. Burası tam olarak ilk satırın değerinin işin içine girdiği yer (yani `function __() { this.constructor = d; }`). Burada etkili şekilde `d.prototype = {__proto__ : __.prototype, d.constructor = d}`'a (çağrılan fonksiyon içerisinde `new`'ın `this` üzerindeki etkisi yüzünden) sahip olacağız. Yani, `d.prototype.constructor`'i eski haline getirdiğimizden beri, gerçekten değiştirdiğimiz tek şey `__proto__`, bu nedenle `d.prototype.__proto__ = b.prototype`.

#### `d.prototype.__proto__ = b.prototype` değeri

Değer şu ki, bu sizin fonksiyon elemanlarınızı alt sınıfa eklemenize ve diğerlerini üst sınıflardan almanıza izin verir. Bunun gösterimi aşağıdaki basit örnekte yapılmıştır:

```ts
function Animal() { }
Animal.prototype.walk = function () { console.log('walk') };

function Bird() { }
Bird.prototype.__proto__ = Animal.prototype;
Bird.prototype.fly = function () { console.log('fly') };

var bird = new Bird();
bird.walk();
bird.fly();
```
Aslında `bird.fly` `bird.__proto__.fly`'da aranacaktır (remember that `new`'ın `bird.__proto__`'nun `Bird.prototype`'u işaret ettireceğini hatırlayın) ve `bird.walk` (türetilen eleman) `bird.__proto__.__proto__.walk`'da aranacaktır (`bird.__proto__ == Bird.prototype` ve `bird.__proto__.__proto__` == `Animal.prototype` gibi).
