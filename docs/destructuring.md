### Parçalama (*destructing*)

TypeScript aşağıda yer alan parçalama biçimlerini destekler:

1. Nesne Parçalama
2. Dizi Parçalama

Parçalamayı oluşturmanın tersi olarak düşünebilirsiniz. JavaScript'te nesneleri oluşturmanın yolu nesne sabitlerini (*object literal*) kullanmaktır;

```ts
var foo = {
    bar: {
        bas: 123
    }
};
```
Eğer JavaScript'te bu şekildeki *oluşturma* yöntemi olmasaydı, anında yeni nesneler yaratmak oldukca zahmetli ve hantal olurdu.
Parçalama işlemi de oluşturma ile aynı kullanım kolaylığı ile gerçekleştirilmektedir.

#### Nesne Parçalama
Parçalama işlemi tek satırlık bir kod ile uygulanabildiği için kullanışlı ve kolaydır. Aşağıdaki örneği inceleyelim:

```ts
var rect = { x: 0, y: 10, width: 15, height: 20 };

// Parçalama ifadesi
var {x, y, width, height} = rect;
console.log(x, y, width, height); // 0,10,15,20
```
Eğer burada parçalama ifadesi olmasaydı, `x,y,width,height` değişkenlerini `rect` üzerinden teker teker çıkarırdık.

Çıkarılan değişkeni yeni bir değişkene atamak aşağıdaki şekilde gerçekleştirilir:

```ts
// oluşturma
const obj = {"some property": "some value"};

// parçalama
const {"some property": someProperty} = obj;
console.log(someProperty === "some value"); // true
```

Buna ek olarak bir yapının *derin*lerindeki veriye erişmek için parçalama işleminden faydalanabilirsiniz. Aşağıdaki örneği inceleyelim:

```ts
var foo = { bar: { bas: 123 } };
var {bar: {bas}} = foo; // Etkin olarak `var bas = foo.bar.bas;`
```

#### Dizi Parçalama
Yaygın bir programlama sorusu: "İki değişkenin değeri üçüncü bir değişken kullanmadan nasıl değiştirilir?". TypeScript çözümü:

```ts
var x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2,1
```
Buradaki önemli nokta şu; derleyici dizi parçalamayı `[0], [1], ...` şeklinde yaptığından dolayı bu değerlerin var olduğunun bir garantisi bulunmuyor. 


#### Dizi Parçalama ve Kalanlar
Bir diziden istediğiniz sayıda elemanı alıp, kalanları da yine *bir dizi* formunda alabilirsiniz.

```ts
var [x, y, ...remaining] = [1, 2, 3, 4];
console.log(x, y, remaining); // 1, 2, [3,4]
```

#### Dizi Parçalama ve Atlananlar
Bir dizi içerisindeki elemanı atlamak için index değerini `, ,` şeklinde boş bırakarak örnekteki gibi ilerleyebilirsiniz:

```ts
var [x, , ...remaining] = [1, 2, 3, 4];
console.log(x, remaining); // 1, [3,4]
```

#### JS Oluşturulması
ES6 dışında kalan ortamda JavaScript oluşturulması basitçe geçici değişkenler ile genel uygulanan yaklaşımdaki gibi gerçekleştirilir.

```ts
var x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2,1

// aşağıdaki gibi dönüştürülür //

var x = 1, y = 2;
_a = [y,x], x = _a[0], y = _a[1];
console.log(x, y);
var _a;
```

#### Özet
Parçalama yaklaşımı, yazılan kod miktarını azaltarak, kod okunabilirliğini arttırır ve bakımını kolaylaştırır. Dizi parçalama işlemi de dizileri birer `tuple` gibi kullanılmasına olanak sağlar.
