### Sınıflar
JavaScript'teki yüksek öncelikli öğeler olan sınıflara sahip olmak şu nedenlerden ötürü önemlidir:
1. [Sınıflar, soyutlama için faydalı bir yapı sunarlar.](./tips/classesAreUseful.md)
2. Geliştiricilere, kendi sürümleriyle gelen frameworkler (emberjs,reactjs etc) yerine, sınıfları kullanabilmeleri için tutarlı bir yöntem sağlar. 
3. Nesneye yönelik geliştiriciler halihazırda sınıf kavramına aşinalar.

Sonunda JavaScript geliştiricileri de artık *`class`*'ları kullanabilirler. Point adında basit bir örnek sınıfımız var: 
```ts
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```
Bu sınıf, ES5'te aşağıdaki gibi bir JavaScript'i üretir:
```ts
var Point = (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.add = function (point) {
        return new Point(this.x + point.x, this.y + point.y);
    };
    return Point;
})();
```
Şimdi bu, birinci sınıf dil yapısı olarak yeterince geleneksel bir JavaScript sınıfı kalıbıdır.

### Kalıtım
Aşağıda gösterildiği gibi, TypeScript'teki sınıflar `extends` anahtar sözcüğü ile *tekil* kalıtımı destekler (diğer diller gibi):

```ts
class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```
Eğer sınıfınızda yapıcı (constructor) metodunuz varsa, yapıcı metodunuzda üst sınıfınızın yapıcı metodunu çağırmak *zorundasınız* (TypeScript bunu size belirtecektir). Bu `this` üzerinde atanması gereken şeylerin atanmasını sağlar. Sonrasında, `super`'in çağırılmasıyla yapıcı metodunuzda yapmak istediğiniz şeyleri ekleyebilirsiniz (Burada `z` adında başka bir eleman ekliyoruz).

Üst eleman fonksiyonlarını kolayca geçersiz kılabileceğinizi (`add`'i burada geçersiz kılıyoruz) ve hala elemanlarınızdaki üst sınıflarınızın işlevselliklerini kullanabileceğinizi göz önünde bulundurun (`super.` sözdizimi kullanarak).

### Statikler (Static)
TypeScript sınıfları, sınıfın tüm örnekleri tarafından paylaşılan `static` özellikleri destekler.Statik özellikleri koymak (ve erişmek) için en doğal yer sınıfın içerisidir ve TypeScript'in yaptığı şey şöyledir:

```ts
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```

Statik fonksiyonlarınız olduğu gibi statik elemanlarınız da olabilir.

### Erişim Belirleyiciler
TypeScript, aşağıda gösterildiği gibi bir `sınıf` elemanının erişilebilirliğini belirleyen `public`,`private` and `protected` erişim belirleyicilerini destekler.

| erişilebilir    | `public` | `protected` | `private` |
|-----------------|----------|-------------|-----------|
| sınıf           | evet     | evet        | evet      |
| alt sınıf       | evet     | evet        | hayır     |
| sınıf örneği    | evet     | hayır       | hayır     |


Eğer bir erişim belirleyici tanımlanmazsa, o JavaScript'in *kullanışlı* doğasına uyduğu için `public` olarak üstü kapalı şekilde belirlenir 🌹.

Çalışma zamanında (üretilen JS'te) bunlar önemsizdir, ancak hatalı kullanırsanız derleme zamanı hataları verecektir. Her biri için örnek gösterim aşağıdadır:

```ts
class FooBase {
    public x: number;
    private y: number;
    protected z: number;
}

// ÖRNEKLERDEKİ ETKİ
var foo = new FooBase();
foo.x; // okay
foo.y; // ERROR : private
foo.z; // ERROR : protected

// ALT SINIFLARDAKİ ETKİ
class FooChild extends FooBase {
    constructor() {
      super();
        this.x; // okay
        this.y; // ERROR: private
        this.z; // okay
    }
}
```

Her zaman olduğu gibi, bu belirleyiciler hem eleman özellikleri hem de eleman fonksiyonları için çalışır.

### Abstract (Soyut)
`abstract` erişim belirleyici olarak düşünülebilir. Ayrı olarak ele almaktayız, çünkü daha önce bahsedilen belirleyicilerle birlikte bir `sınıfın` yanı sıra, sınıfın herhangi bir elemanı da olabilir. `abstract` bir belirleyiciye sahip olmak, öncelikle bu işlevselliği *doğrudan çağrılamaz* ve bir alt sınıfı fonksiyonları sağlamalıdır demektir.

* `abstract` **sınıflar** doğrudan örneklenemezler. Bunun yerine kullanıcı `abstract sınıf`'tan türemiş `sınıf` yaratmalıdır.
* `abstract` **elemanlar** doğrudan erişilemez ve alt sınıf, fonksiyonları sağlamalıdır.

### Yapıcı Metot (Constructor) İsteğe Bağlıdır

Sınıfın yapıcı metoda sahip olması gerekmez. Sıradaki örnek bunun için güzel bir örnektir. 

```ts
class Foo {}
var foo = new Foo();
```

### Yapıcı Metot kullanım tanımı

Sınıfta bir elemana sahip olmak ve onu aşağıdaki gibi yüklenmesi:

```ts
class Foo {
    x: number;
    constructor(x:number) {
        this.x = x;
    }
}
```
bu TypeScript'in elemanı *erişim belirleyici* ile ön ek olarak sağladığı yaygın bir kalıptır, sınıf üzerinde otomatik olarak deklare edilir ve yapıcı metotdan kopyalanır. Yani önceki örnek şu şekilde tekrar yazılabilir (dikkat `public x:number`):

```ts
class Foo {
    constructor(public x:number) {
    }
}
```

### Özellik yükleyici
Bu TypeScript tarafından desteklenen havalı bir özelliktir (ES7'nin aslında). Sınıfın yapıcı metodu dışında herhangi bir sınıf elemanını yükleyebilirsiniz, varsayılanı sağlamak için kullanışlıdır (dikkat `members = []`)

```ts
class Foo {
    members = [];  // Direk olarak yüklenir
    add(x) {
        this.members.push(x);
    }
}
```
