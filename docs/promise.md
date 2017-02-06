## Promise

Promise, pek çok modern Javascript motorunda bulunan ve rahatlıkla polyfill edilebilen bir sınıftır. Promise'lerin öncelikli amacı Async/Callback tarzı yazılmış koda senkronize stilde hata yakalama fonksiyonunu kazandırmaktır.

### Callback tarzı kod

Promise'in sağladığı kolaylıkları daha iyi anlamak için sadece Callback kullanarak Async çalışan bir örnek görelim. Bir JSON dosyasından Async bir şekilde dosya okuma örneğini değerlendirelim. Sync bir versiyonu oldukça kolay olacaktır:

```ts
import fs = require('fs');

function loadJSONSync(filename: string) {
    return JSON.parse(fs.readFileSync(filename));
}

// geçerli json dosyası
console.log(loadJSONSync('good.json'));

// var olmayan json dosyası. bu yüzden fs.readFilesync hata verir
try {
    console.log(loadJSONSync('absent.json'));
}
catch (err) {
    console.log('absent.json error', err.message);
}

// geçersiz json dosyası. dosya var ama içindeki JSON geçersiz. bu yüzden JSON.parse hata verir
try {
    console.log(loadJSONSync('invalid.json'));
}
catch (err) {
    console.log('invalid.json error', err.message);
}
```

Bu basit `loadJSONSync` fonksiyonunun üç davranışı vardır. Geçerli bir dönüş değeri, bir dosya sistemi hatası ya da JSON.parse hatası. Bu hataları diğer Sync çalışan dillerde yaptığımız gibi basit try/catch bloğu ile yakalıyoruz. Şimdi bu fonksiyonun düzgün çalışan bir Async versiyonunu yapalım. Düzgün bir ilk deneme (küçük bir hata yakalama mekanizması ile) aşağıdaki şekilde olacaktır.

```ts
import fs = require('fs');

// uygun bir ilk deneme... ama yanlış çalışıyor. Sebeplerini aşağıda açıklayacağız.
function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}
```

Yeteri kadar basit, bir callback alır, bulduğu dosya sistemi hatalarını callback'e iletir. Eğer dosya sistemi hatası yok ise JSON.parse işleminin sonucunu döndürür. Callback'lere dayalı Async fonksiyonlarla çalışılırken unutulmaması gereken noktalar:

1. Bir callback'i asla iki defa çağırmamak
2. Asla hata fırlatmamak

Bu basit fonksiyon 2. noktada problem yaşıyor. Aslında eğer geçersiz JSON verilir ise, JSON.parse hata verir, callback hiçbir zaman çağırılmaz ve uygulama çöker. Bu durum aşağıdaki örnekte gösterilmiştir.

```ts
import fs = require('fs');

// uygun bir ilk deneme ama çalışmıyor.
function loadJSON(filename: string, cb: (error: Error, data: any) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) cb(err);
        else cb(null, JSON.parse(data));
    });
}

// hatalı json yüklüyoruz
loadJSON('invalid.json', function (err, data) {
    // This code never executes
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

Bu durumu düzeltmek için naifçe bir çaba, JSON.parse'ı bir try/catch'e almak olurdu. Aşağıdaki örnekteki gibi:

```ts
import fs = require('fs');

// daha iyi bir deneme ama hala hatalı
function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// hatalı json yüklüyoruz
loadJSON('invalid.json', function (err, data) {
    if (err) console.log('bad.json error', err.message);
    else console.log(data);
});
```

Yine de bu kodda yakalaması zor bir hata var. Eğer `JSON.parse` değil de callback(`cb`) hata fırlatırsa, biz bunu `try`/`catch` ile sarmaladığımız için `catch` çalışır ve callback'i bir daha çağırırız. Bu örnekte callback iki defa çağırılır! Bu, aşağıdaki örnekte gösterilmiştir:

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) {
            cb(err);
        }
        else {
            try {
                cb(null, JSON.parse(data));
            }
            catch (err) {
                cb(err);
            }
        }
    });
}

// düzgün bir dosya ama hatalı bir callback... tekrar çağırılıyor
loadJSON('good.json', function (err, data) {
    console.log('callback çağırıldı');

    if (err) console.log('Error:', err.message);
    else {
        // atanmamış(undefined) bir değerdeki bir alana erişmeye çalışarak bir hata simüle ediyoruz
        var foo;
        // aşağıdaki kod `Error: Cannot read property 'bar' of undefined` hatası verecektir
        console.log(foo.bar);
    }
});
```

```bash(konsol)
$ node asyncbadcatchdemo.js
callback çağırıldı
callback çağırıldı
Error: Cannot read property 'bar' of undefined
```

Bunun sebebi `loadJSON` fonksiyonumuzun yanlış biçimde callback'i bir `try` bloğu ile sarmalamış olmasıdır. Burada hatırlanması gereken küçük bir ders var.

> küçük Bir Ders: Tüm senkronize kodunuzu bir try/catch içine alın, callback'i çağırdığnız yer hariç.

Bu küçük dersi takip ederek `loadJSON` metodumuzun tamamen fonksiyonel asenkron çalışan bir versiyonunu elde ediyoruz:

```ts
import fs = require('fs');

function loadJSON(filename: string, cb: (error: Error) => void) {
    fs.readFile(filename, function (err, data) {
        if (err) return cb(err);
        // senkronize çalışması gereken tüm kodu try'ın içine alın
        try {
            var parsed = JSON.parse(data);
        }
        catch (err) {
            return cb(err);
        }
        // callback'i çağırdığınız an hariç
        return cb(null, parsed);
    });
}
```
Bu, birkaç kez yaptıktan sonra çok daha kolay olmakla beraber, basit bir hata yakalama için çok fazla boilerplate kod yazmak anlamına geliyor. Şimdi promise'leri kullanarak javascript'te asenkron kodla uğraşmanın daha iyi bir yolunu bulalım.


## Bir Promise oluşturmak

Bir promise `pending`(çalışmaya devam ediyor), `resolved`(çalışması sonlanmış) veya `rejected`(reddedilmiş) durumunda olabilir.

![](https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png)

Bir promise oluşturalım. Bunun için promise yapıcı metodu (constructor) üzerinde `new` kelimesini çalıştırmak yeterli. `resolve` and `reject` fonksiyonları promise'in durumunu almak için yapıcı metoda geçilir.

```ts
const promise = new Promise((resolve, reject) => {
    // resolve / reject fonksiyonları promise'in sonucunu belirler
});
```

### Promise'in sonucunu gözlemlemek

Promise'in sonucu `.then`(eğer sonlanmış ise) veya `.catch`(eğer reddedilmiş ise) metotları ile gözlemlenebilir.

```ts
const promise = new Promise((resolve, reject) => {
    resolve(123);
});
promise.then((res) => {
    console.log('I get called:', res === 123); // Ben çağırıldım. Sonuç: true
});
promise.catch((err) => {
    // Burası hiç çağırılmadı
});
```

```ts
const promise = new Promise((resolve, reject) => {
    reject(new Error("Çok kötü bir şey oldu."));
});
promise.then((res) => {
    // Burası hiç çağırılmadı
});
promise.catch((err) => {
    console.log('Çağırıldım :', err.message); // Çağırıldım : "Çok kötü bir şey oldu"
});
```

> İPUCU: Promise Kısayolları
* Hızlı bir şekilde çözülmüş bir promise yaratmak : `Promise.resolve(result)`
* Hızlı bir şekilde reddedilmiş bir promise yaratmak : `Promise.reject(error)`

### Promise'lerin zincirlenebilmesi
Promise'lerin zincirlenebilir olması **en önemli özelliğidir**. Bir Promise'iniz olduğunda, `then` fonksiyonunu kullanarak promise zinciri yaratabilirsiniz.

* Zincirdeki herhangi bir fonksiyondan bir promise döndürürseniz, `.then` sadece fonksiyon çözümlendiğinde çağırılır.

```ts
Promise.resolve(123)
    .then((res) => {
        console.log(res); // 123
        return 456;
    })
    .then((res) => {
        console.log(res); // 456
        return Promise.resolve(123); // Bir promise döndürüyor olduğumuza dikkat edin
    })
    .then((res) => {
        console.log(res); // Bu `then`'in çözümlenmiş değer ile çağırıldığına dikkat edin.   
        return 123;
    })
```

* zincirin herhangi bir kısmında gerçekleşen bir hatayı tek bir `catch` ile yakalayabilirsiniz.

```ts
// Reddedilen bir promise yaratın
Promise.reject(new Error('kötü bir şey oldu'))
    .then((res) => {
        console.log(res); // çağırılmadı
        return 456;
    })
    .then((res) => {
        console.log(res); // çağırılmadı
        return 123;
    })
    .then((res) => {
        console.log(res); // çağırılmadı
        return 123;
    })
    .catch((err) => {
        console.log(err.message); // kötü bir şey oldu
    });
```

* `catch` yeni bir promise döndürür(yeni bir promise zinciri yaratarak)

```ts
// Reddedilecek bir promise yaratalım
Promise.reject(new Error('kötü bir şey oldu'))
    .then((res) => {
        console.log(res); // çağırılmadı
        return 456;
    })
    .catch((err) => {
        console.log(err.message); // kötü bir şey oldu
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
    })
```

* Bir `then` (ya da `catch`) fonksiyonunda gerçekleşen herhangi bir asenkron hata, döndürülen promise'in fail olmasına sebep olur.

```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('kötü bir şey oldu'); // asenkron bir hata fırlatalım
        return 456;
    })
    .then((res) => {
        console.log(res); // hiç çağırılmadı
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log(err.message); // kötü bir şey oldu
    })
```

Gerçek şu ki:

* hatalar, sıradaki ilk `catch`'e gider (aradaki `then`'leri atlayarak) ve
* senkronizasyon hatası da sıradaki ilk `catch` ile yakalanır

Bu da, bize sadece callback kullanımına kıyasla daha iyi bir hata yakalama sağlayan, efektif bir asenkron programlama paradigması kazandırır. Bu örnek üzerine daha fazla bilgi aşağıdadır.


### TypeScript ve Promise'ler
Typescript ile ilgili harika olan şey, bir promise chain içerisinde gerçekleşen değer akışını anlayabilmesidir.

```ts
Promise.resolve(123)
    .then((res)=>{
         // res'in `number` olduğuna karar verilir
         return true;
    })
    .then((res) => {
        // res'in `boolean` tipinde olduğuna karar verilir

    });
```

Bu yapı tabii ki promise döndürme ihtimali olan fonksiyon çağrılarını da anlar:


```ts
function iReturnPromiseAfter1Second():Promise<string> {
    return new Promise((resolve)=>{
        setTimeout(()=>resolve("Merhaba Dünya!"), 1000);
    });
}

Promise.resolve(123)
    .then((res)=>{
         // res'in `number` olduğuna karar verilir
         return iReturnPromiseAfter1Second(); // Bir promise döndürüyoruz `Promise<string>`
    })
    .then((res) => {
        // res'in `string` olduğuna karar verilir
        console.log(res); // Merhaba Dünya!
    });
```


### Callback tarzı yazılmış bir fonksiyonu Promise döndüren bir fonksiyona çevirmek

Fonksiyon çağırımını bir promise ile sarmalayın ve
- herhangi bir hata olursa `reject`,
- olmazsa `resolve` döndürün.

Örnek olarak `fs.readFile` sarmalayalım:

```ts
import fs = require('fs');
function readFileAsync(filename:string):Promise<any> {
    return new Promise((resolve,reject)=>{
        fs.readFile(filename,(err,result) => {
            if (err) reject(err);
            else resolve(result);
        });
    });
}
```


### JSON örneğine geri dönüş

Şimdi `loadJSON` örneğimize geri dönelim ve promise'leri kullanan async bir versiyonunu yazalım. Yapmamız gereken tek şey dosya içeriğini bir promise olarak okumak ve okuma bittiğinde JSON olarak parse etmek. Bu, aşağıdaki örnekte gösterilmiştir:

```ts
function loadJSONAsync(filename: string): Promise<any> {
    return readFileAsync(filename) // yazmış olduğumuz fonksiyonu kullanıyoruz
                .then(function (res) {
                    return JSON.parse(res);
                });
}
```

Kullanım (bu bölümün başında yapmış olduğumuz `sync` versiyona ne kadar benzediğine dikkat edin 🌹):
```ts
// geçerli json dosyası
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // hiç çağırılmadı
    })

// var olmayan json dosyası
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // hiç çağırılmadı
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })

// invalid json file
    .then(function () {
        return loadJSONAsync('invalid.json');
    })
    .then(function (val) { console.log(val); }) // hiç çağırılmadı
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```

Bu fonksiyonun daha basit olmasının sebebi "`loadFile`(async) + `JSON.parse` (sync) => `catch`" kısmının promise zinciri tarafından üstlenmilmiş olmasıdır. Ayrıca callback *bizim tarafımızdan değil*, promise zinciri tarafından çağırıldığı için `try/catch` bloğu ile sarmalama hatasına düşmemiş olduk.

### Paralel akış kontrolü
Promise kullanarak asenkron işler yapmanın ne kadar kolay olduğunu gördük. Aslında bu sadece `then` çağırımlarını birbirine bağlamaktan ibaretti.

Yine de birden fazla asenkron işi gerçekleştirip aldığınız sonuçla başka bir iş yapmak isteyebilirsiniz. `Promise`, vermiş olduğunuz `n` sayıdaki promise'in tamamlanmasını bekleyip, toplam sonucu döndüren statik bir `Promise.all` fonksiyonuna sahiptir. Bu fonksiyona `n` sayıda promise içeren bir dizi verirsiniz ve bu fonksiyon da size `n` sayıda çözümlenmiş sonuç döndürür. Aşağıda paralel'in yanı sıra zincirlemeyi de gösteriyoruz:

```ts
// bir sunucudan, bir nesnenin yüklenmesini simüle eden asenkton bir fonksiyon
function loadItem(id: number): Promise<{id: number}> {
    return new Promise((resolve)=>{
        console.log('loading item', id);
        setTimeout(() => { // sunucu gecikmesini simüle ediyoruz
            resolve({ id: id });
        }, 1000);
    });
}

// zincirleme
let item1, item2;
loadItem(1)
    .then((res) => {
        item1 = res;
        return loadItem(2);
    })
    .then((res) => {
        item2 = res;
        console.log('done');
    }); // toplam geçen zaman 2 saniye civarında olacaktır

// Parallel
Promise.all([loadItem(1),loadItem(2)])
    .then((res) => {
        [item1,item2] = res;
        console.log('done')
    }); //  toplam geçen zaman 1 saniye civarında olacaktır
```


[polyfill]:https://github.com/stefanpenner/es6-promise
