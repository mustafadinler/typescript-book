### Şablon String'ler  
Tek tırnak veya çift tırnak ile tanımlanan string'ler yerine aksan işareti (\`) ile tanımlanmış string'lerdir. Şablon String'ler aşağıdaki amaçlar için kullanılır:

* String Aradeğerleme
* Çok Satırlı String'ler
* Biçimlendirilmiş Şablonlar

#### String Aradeğerleme
Verilen string ile değişkenlerin değerini barındıran bir string çıktısı elde edilmek istendigi durumda yaygın olarak kullanılan bir yöntemdir. Bunun gerçekleştirilmesi için bir *şablonlama mantığı* gereksinimi bulundugundan *şablon string'ler* adını buradan alır.
Sıradan yollarla bir html string tanımlamak istendiğinde büyük olasılıkla şu şekilde gerçekleştirilecektir:

```ts
var lyrics = 'Never gonna give you up';
var html = '<div>' + lyrics + '</div>';
```
Şablon string'ler ile bu şekilde yapılabilir:

```ts
var lyrics = 'Never gonna give you up';
var html = `<div>${lyrics}</div>`;
```

Aradeğerleme işlemi (`${` ve `}`)  içerisindeki yertutucuların birer JavaScript ifadesi olarak işlenip değerlendirileceğini unutmayınız. Örneğin burada bir takım matematiksel işlemler de gerçekleştirilebilir.

```ts
console.log(`1 + 1 = ${1 + 1}`);
```

#### Çok Satırlı String'ler
Hiç JavaScript string'lerinde satırbaşı yapmak istediniz mi? Ya da bir şarkı sözü belirtmek istemediniz mi? Bu durumda string'imizin ilgili yerine favori *escape karakteri*miz olan `\` ile *satırbaşı karakteri* olan `\n`'yi 
aşağıdaki örnekte gösterildiği gibi yerleştirmemiz gerekecektir:

```ts
var lyrics = "Never gonna give you up \
\nNever gonna let you down";
```

TypeScript'teki şablon string'ler aşağıdaki kolaylığı sağlamakta:

```ts
var lyrics = `Never gonna give you up
Never gonna let you down`;
```

#### Biçimlendirilmiş Şablonlar
Şablon string tanımının önüne (`tag` olarak isimlendirilen) fonksiyonları ekleyebilir ve ilgili şablon string'inin ön işlenmesini gerçekleştirerek buradan dönen sonucu kullanabilirsiniz.
Not:
* Şablondaki tüm statik string'ler çağırılacak fonksiyonun ilk parametresine bir dizi olarak iletilir.
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
ES6 öncesi derlemede oluşturulan çıktı oldukça basit. Çok satırlı string'ler escape karakterlilere çevirilir. String aradeğerleme ifadeleri *string birleştirme*ye dönüşür. Biçimlendirilmiş şablonlar normal fonksiyon çağırımları ile değiştirilir.

#### Özet
Bir programlama dilinde *çok satırlı string'ler* ve *string aradeğerleme* kullanımlarının bulunması büyük kolaylık sağlamaktadır. TypeScript sayesinde bunlar artık JavaScript ile kullanılabililir. Biçimlendirilmiş şablonlar ile de detaylı string işlemleri gerçekleştirilebilen uzantılar yazılabilir.
