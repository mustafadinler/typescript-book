## Promise

Promise, pek çok modern Javascript motorunda bulunan ve rahatlıkla polyfill edilebilen bir sınıftır. Promise'lerin öncelikli amacı Async/Callback stili yazılmış koda senkronize stilde hata yakalama fonksiyonunu kazandırmaktır.

### Callback style code

Promise'in sağladığı kolaylıkları daha iyi anlamak için sadece Callback kullanarak Async kod yazan bir örnek görelim. Bir JSON dosyasından asenkron bir şekilde dosya okuma örneğini değerlendirelim. Senkronize bir vesiyonu oldukça kolay olacaktır:

```ts
import fs = require('fs');

function loadJSONSync(filename: string) {
    return JSON.parse(fs.readFileSync(filename));
}

// geçerli json dosyası
console.log(loadJSONSync('good.json'));

// olmayan json dosyası. bu yüzden fs.readFilesync hata verir
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

Bu basit `loadJSONSync` fonksiyonunun üç davranışı vardır. Geçerli bir dönüş değeri, bir dosya sistemi hatası ya da JSON.parse hatası. Bu hataları diğer senkronize çalışan dillerde yaptığımız gibi basit try/catch bloğu ile yakalıyoruz. Şimdi bu fonksiyonun düzgün çalışan bir asenkron versiyonunu yapalım. Düzgün bir ilk deneme (with a trivial error checking) aşağıdaki şekilde olacaktır.

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

Yeteri kadar basit, bir callback alır, bulduğu dosya sistemi hatalarını callback'e iletir. Eğer dosya sistemi hatası yok ise JSON.parse işleminin sonucunu döndürür. Callback'lere dayalı asenkron fonksiyonlarla çalışılırken unutulmaması gereken noktalar:

1. Bir callback'i asla iki defa çağırmamak
2. Asla hata fırlatmamak

Bu basit fonksiyon 2. noktada çalışmakta problem yaşıyor. Aslında eğer geçersiz JSON verilir ise, JSON.parse hata verir, callback hiçbir zaman çağırılmaz ve uygulama çöker. Bu durum aşağıdaki örnekte gösterilmiştir.

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

Bu durumu düzeltmek için naifçe bir çaba, JSON.parse'ı bir try/catch'e almak olurdu, aşağıdaki örnekteki gibi:

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

Yine de bu kodda yakalaması zor bir hata var. Eğer `JSON.parse` değil de callback(`cb`) hata fırlatırsa, biz bunu `try`/`catch` ile sarmaladığımız için `catch` çalışır ve callback'i bir daha çağırırız. Bu örnekte callback iki defa çağırılır! Bu aşağıdaki örnekte gösterilmiştir:

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
Bunu birkaç kez yaptıktan sonra çok daha kolay olmakla beraber basit bir hata yakalama için bu çok fazla boilerplate kod yazmak anlamına geliyor. Şimdi promise'leri kullanarak javascript'te asenkron kodla uğraşmanın daha iyi bir yolunu bulalım.


## Bir Promise oluşturmak

Bir promise `pending`(çalışmaya devam ediyor), `resolved`(çalışması sonlanmış) veya `rejected`(reddedilmiş) durumunda olabilir.

![](https://raw.githubusercontent.com/basarat/typescript-book/master/images/promise%20states%20and%20fates.png)

Bir promise oluşturalım. Bunun için promise ctor'u üzerinde `new` kelimesini çalıştırmak yeterli. `resolve` and `reject` fonksiyonları promise'in durumunu almak için constructor'a geçilir.

```ts
const promise = new Promise((resolve, reject) => {
    // resolve / reject fonksiyonları promise'in sonucunu belirler
});
```

### Promise'in sonucuna abone olmak

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

> TIP: Promise Shortcuts
* Quickly creating an already resolved promise : `Promise.resolve(result)`
* Quickly creating an already rejected promise : `Promise.reject(error)`

### Promise'lerin zincirlenebilmesi
The chain-ability of promises **is the heart of the benefit that promises provide**. Once you have a promise, from that point on, you use the `then` function to create a chain of promises.

* If you return a promise from any function in the chain, `.then` is only called once the value is resolved

```ts
Promise.resolve(123)
    .then((res) => {
        console.log(res); // 123
        return 456;
    })
    .then((res) => {
        console.log(res); // 456
        return Promise.resolve(123); // Notice that we are returning a Promise
    })
    .then((res) => {
        console.log(res); // 123 : Notice that this `then` is called with the resolved value
        return 123;
    })
```

* you can aggregate the error handling of any preceding portion of the chain with a single `catch`

```ts
// Create a rejected promise
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .then((res) => {
        console.log(res); // not called
        return 123;
    })
    .then((res) => {
        console.log(res); // not called
        return 123;
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
    });
```

* the `catch` actually returns a new promise (effectively creating a new promise chain):

```ts
// Create a rejected promise
Promise.reject(new Error('something bad happened'))
    .then((res) => {
        console.log(res); // not called
        return 456;
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
        return 123;
    })
    .then((res) => {
        console.log(res); // 123
    })
```

* Any synchronous errors thrown in a `then` (or `catch`) result in the returned promise to fail

```ts
Promise.resolve(123)
    .then((res) => {
        throw new Error('something bad happened'); // throw a synchronous error
        return 456;
    })
    .then((res) => {
        console.log(res); // never called
        return Promise.resolve(789);
    })
    .catch((err) => {
        console.log(err.message); // something bad happened
    })
```

The fact that:

* errors jump to the tailing `catch` (and skip any middle `then` calls) and
* synchronous errors also get caught by any tailing `catch`.

effectively provides us with an async programming paradigm that allows better error handling than raw callbacks. More on this below.


### TypeScript and promises
The great thing about TypeScript is that it understands the flow of values through a promise chain.

```ts
Promise.resolve(123)
    .then((res)=>{
         // res is inferred to be of type `number`
         return true;
    })
    .then((res) => {
        // res is inferred to be of type `boolean`

    });
```

Of course it also understands unwrapping any function calls that might return a promise:

```ts
function iReturnPromiseAfter1Second():Promise<string> {
    return new Promise((resolve)=>{
        setTimeout(()=>resolve("Hello world!"), 1000);
    });
}

Promise.resolve(123)
    .then((res)=>{
         // res is inferred to be of type `number`
         return iReturnPromiseAfter1Second(); // We are returning `Promise<string>`
    })
    .then((res) => {
        // res is inferred to be of type `string`
        console.log(res); // Hello world!
    });
```


### Converting a callback style function to return a promise

Just wrap the function call in a promise and
- `reject` if an error occurs,
- `resolve` if it is all good.

E.g. let's wrap `fs.readFile`

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


### Revisiting the JSON example

Now let's revisit our `loadJSON` example and rewrite an async version that uses promises. All that we need to do is read the file contents as a promise, then parse them as JSON and we are done. This is illustrated in the below example:

```ts
function loadJSONAsync(filename: string): Promise<any> {
    return readFileAsync(filename) // Use the function we just wrote
                .then(function (res) {
                    return JSON.parse(res);
                });
}
```

Usage (notice how similar it is to the original `sync` version introduced at the start of this section 🌹):
```ts
// good json file
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // never called
    })

// non-existent json file
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })

// invalid json file
    .then(function () {
        return loadJSONAsync('invalid.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```

The reason why this function was simpler is because the "`loadFile`(async) + `JSON.parse` (sync) => `catch`" consolidation was done by the promise chain. Also the callback was not called by *us* but called by the promise chain so we didn't have the chance of making the mistake of wrapping it in a `try/catch`.

### Parallel control flow
We have seen how trivial doing a serial sequence of async tasks is with promises. It is simply a matter of chaining `then` calls.

However you might potentially want to run a series of async tasks and then do something with the results of all of these tasks. `Promise` provides a static `Promise.all` function that you can use to wait for `n` number of promises to complete. You provide it with an array of `n` promises and it gives you array of `n` resolved values. Below we show Chaining as well as Parallel:

```ts
// an async function to simulate loading an item from some server
function loadItem(id: number): Promise<{id: number}> {
    return new Promise((resolve)=>{
        console.log('loading item', id);
        setTimeout(() => { // simulate a server delay
            resolve({ id: id });
        }, 1000);
    });
}

// Chaining
let item1, item2;
loadItem(1)
    .then((res) => {
        item1 = res;
        return loadItem(2);
    })
    .then((res) => {
        item2 = res;
        console.log('done');
    }); // overall time will be around 2s

// Parallel
Promise.all([loadItem(1),loadItem(2)])
    .then((res) => {
        [item1,item2] = res;
        console.log('done')
    }); // overall time will be around 1s
```


[polyfill]:https://github.com/stefanpenner/es6-promise
