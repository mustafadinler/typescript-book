### Şablon Stringler  
Tek tırnak veya çift tırnak ile tanımlanan stringler yerine aksan işareti (\`) ile tanımlanmış stringlerdir. Şablon Stringler aşağıdaki amaçlar için kullanılır:

* String Aradeğerleme
* Çok Satırlı Stringler
* Biçimlendirilmiş Şablonlar

#### String Aradeğerleme
Verilen string ile değişkenlerin değerini barındıran bir string çıktısı elde edilmek istendigi durumda yaygın olarak kullanılan bir yöntemdir. Bunun gerçekleştirilmesi için bir *şablonlama mantığı* gereksinimi bulundugundan *şablon stringler* adını buradan alır.
Sıradan yollarla bir html string tanımlamak istendiğinde büyük olasılıkla şu şekilde gerçekleştirilecektir:

```ts
var lyrics = 'Never gonna give you up';
var html = '<div>' + lyrics + '</div>';
```
Şablon stringler ile bu şekilde yapılabilir:

```ts
var lyrics = 'Never gonna give you up';
var html = `<div>${lyrics}</div>`;
```

Aradeğerleme işlemi (`${` ve `}`)  içerisindeki yertutucuların birer JavaScript ifadesi olarak işlenip değerlendirileceğini unutmayınız. Örneğin burada bir takım matematiksel işlemler de gerçekleştirilebilir.

```ts
console.log(`1 + 1 = ${1 + 1}`);
```

#### Çok Satırlı Stringler
Hiç JavaScript stringlerinde satırbaşı yapmak istediniz mi? Ya da bir şarkı sözü belirtmek istemediniz mi? Bu durumda stringimizin ilgili yerine favori *kaçış karakteri*miz olan `\` ile *satırbaşı karakteri* olan `\n`'yi 
aşağıdaki örnekte gösterildiği gibi yerleştirmemiz gerekecektir:

```ts
var lyrics = "Never gonna give you up \
\nNever gonna let you down";
```

TypeScript'teki şablon stringler aşağıdaki kolaylığı sağlamakta:

```ts
var lyrics = `Never gonna give you up
Never gonna let you down`;
```

#### Biçimlendirilmiş Şablonlar
Şablon string tanımının önüne (`tag` olarak isimlendirilen) fonksiyonları ekleyebilir ve ilgili şablon stringinin ön işlenmesini gerçekleştirerek buradan dönen sonucu kullanabilirsiniz.
Not:
* Şablondaki tüm statik stringler çağırılacak fonksiyonun ilk parametresine bir dizi olarak iletilir.
* Kalan tüm yertutucu ifadelerin değerleri fonksiyonun diğer argümanlarını oluşturur. Genellikle sonradan iletilen tüm bu parametreleri bir diziye çevirerek kullanım sağlanır. 

Verilen html içerisinden tüm yertutucuların arındırıldığı `htmlEscape` adıyla tanımlmanan bir biçimlendirme fonksiyonunun hazırlanması ve kullanımı aşağıdaki gibidir:

```ts
var say = "a bird in hand > two in the bush";
var html = htmlEscape `<div> I would just like to say : ${say}</div>`;

// örnek biçimlendirme fonksiyonu
function htmlEscape(literals, ...placeholders) {
    let result = "";

    // yertutucularin değerler ile değiştirilmesi
    for (let i = 0; i < placeholders.length; i++) {
        result += literals[i];
        result += placeholders[i]
            .replace(/&/g, '&amp;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#39;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;');
    }

    // son değerin eklenmesi
    result += literals[literals.length - 1];
    return result;
}
```

#### Üretilen JS
ES6 öncesi derlemede oluşturulan çıktı oldukça basit. Çok satırlı stringler kaçış karakterlilere çevirilir. String aradeğerleme ifadeleri *string birleştirme*ye dönüşür. Biçimlendirilmiş şablonlar normal fonksiyon çağırımları ile değiştirilir.

#### Özet
Bir programlama dilinde *çok satırlı stringler* ve *string aradeğerleme* kullanımlarının bulunması buyuk kolaylık sağlamaktadır. TypeScript sayesinde bunlar artık JavaScript ile kullanılabililir. Biçimlendirilmiş şablonlar ile de detaylı string işlemleri gerçekleştirilebilen uzantılar yazılabilir.
