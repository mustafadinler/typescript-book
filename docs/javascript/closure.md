## Kapsanım

JavaScript'in sahip olduğu en iyi şey kapsanımlardır. JavaScript'de bir fonksiyon dış faaliyet alanında tanımlanmış herhangi bir değişkene erişebilir. Kapsanımlar en iyi şekilde örneklerle açıklanabilir:

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;

    function bar() {
        console.log(variableInOuterFunction); // Dış faaliyet alanındaki bir değişkene erişim 
    }

    // arg'ye erişim olduğunu göstermek için yerel fonksiyon çağrısı
    bar();
}

outerFunction("merhaba kapsanım"); // merhaba kapsanım çıktısı üretir!
```

Gördüğünüz üzere iç fonksiyonun, dış faaliyet alanındaki değişkene (variableInOuterFunction) erişimi vardır. Dış fonksiyondaki değişkenler iç fonksiyon tarafından kapatılmıştır (veya bağlanmıştır). Bu nedenle terim **kapsanım**. Kendi içindeki kavram yeterince basit ve oldukça sezgisel.

Şimdi en harika kısmı: *Dış fonksiyon geri döndükten sonra bile* iç fonksiyon, dış faaliyet alanındaki değişkenlere erişebilir. Bunun nedeni, değişkenlerin hala iç fonksiyona bağlı olması ve dış fonksiyona bağımlı olmamasıdır. Gelin tekrar örneğe bakalım:

```ts
function outerFunction(arg) {
    var variableInOuterFunction = arg;
    return function() {
        console.log(variableInOuterFunction);
    }
}

var innerFunction = outerFunction("merhaba kapsanım!");

// outerFunction geri geldiğine dikkat edin
innerFunction(); // merhaba kapsanım! çıktısı üretir
```

### Harika olmasının sebebi
Size nesneleri kolayca oluşturmanızı sağlar, örneğin açıklayıcı modül kalıbı:

```ts
function createCounter() {
    let val = 0;
    return {
        increment() { val++ },
        getVal() { return val }
    }
}

let counter = createCounter();
counter.increment();
console.log(counter.getVal()); // 1
counter.increment();
console.log(counter.getVal()); // 2
```

Üst seviyede nodejs gibi bir şeyi mümkün kılan şey de budur (Kafanızda şekillenmediyse endişelenmeyin. Sonunda olacak 🌹):

```ts
// Kavramı açıklamak için sözde kod (pseudo code)
server.on(function handler(req, res) {
    loadData(req.id).then(function(data) {
        // `res` kapatıldı ve hazır
        res.send(data);
    })
});
```
