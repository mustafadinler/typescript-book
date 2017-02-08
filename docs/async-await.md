## Asenkron Bekleme
Şöyle bir deney kurguladığımızı varsayalım, Javascript çalışma zamanında `await` anahtar kelimesi bir `promise`(söz) ile birlikte görüldüğü anda kodun çalışması duraksayabilir ve sadece verilen sözün sonuca ulaşması tamamlandıktan sonra -sadece bir kerelik- çalışmaya kaldığı yerden devam eder.

```ts
//Aşağıda gördüğünüz gerçek bir kod parçası değildir
async function foo() {
    try {
        var val = await getMeAPromise();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
}

```
Verilen söz bir sonuca ulaştığında program çalışmaya devam eder,
* Eğer verilen söz yerine getirildiyse `await` verilen sözün sonucunu dönecektir,
* Eğer verilen söz yerine getirilmediyse (reddedildiyse), çalışma zamanında yakalayabileceğimiz bir hata senkron olarak fırlatılacaktır  

Bu yaklaşım asenkron programlamayı bir anda senkron programlama gibi kolay bir hale getiriyor.Bu deney için üç bileşen gereklidir.  
* *Fonksiyonları duraksatabilme* kabiliyeti.
* Fonksiyonun içerisine *bir değer koyabilme* kabiliyeti.
* Fonksiyonun içerisinden *bir istisna fırlatabilme* kabiliyeti.

İşte bu saydıklarımız tam olarak *oluşturucular*ın yapmamıza olanak tanıdığı şey. Aslında bahsettiğimiz deney bir kurgu değil gerçeğin ta kendisidir ve  Javascript / Typescript dünyasında `asenkron`/`bekle` uyarlaması olarak ifade edilir.Bu uyarlama perdenin arkasında oluşturucuları kullanır.


### Oluşturulmuş JavaScript
Bunu tamamen anlamak zorunda değilsiniz ama [oluşturucuları][generators] okursanız oldukça basit olduğunu göreceksiniz. `foo` fonksiyonu aşağıdaki gibi basit bir şekilde paketlenebilir:

```ts
const foo = sozAlmakicinPaketle(function* () {
    try {
        var val = yield banaSozVer();
        console.log(val);
    }
    catch(err) {
        console.log('Error: ', err.message);
    }
})
```
`sozAlmakicinPaketle`'nin `oluşturucuyu` almak için sadece oluşturucu fonksiyonu çağırdığı bir durumda; daha sonra `generator.next()` ifadesi çağırılır, eğer dönen değer bir `söz` ise, `generator.next(result)` veya `generator.throw(error)` ifadelerinden biri duruma göre `then` veya `catch` ile yakalanarak işlem sonlandırılır. 


[generators]:./generators.md
