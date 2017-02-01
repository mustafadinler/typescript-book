# JavaScript'iniz TypeScript'tir.

Bazı sözdiziminden JavaScript derleyicilerine kadar birçok rakip vardı (ve olmaya devam edecektir). TypeScript onlardan şu şekilde farklıdır *JavaScript'iniz TypeScript'tir*. İşte bir şema:

![](https://raw.githubusercontent.com/basarat/typescript-book/master/images/venn.png)

Ancak bunun anlamı *JavaScript öğrenmeniz gerekir* (iyi haber şu ki *sizin **sadece** JavaScript öğrenmeniz gerekmektedir*). TypeScript sadece JavaScript'teki sunulan *iyi dökümantasyon'u* standartlaştırmaktadır.

* Size yeni bir sözdizimi verilmesi hataların düzeltilmesine yardımcı olmaz (CoffeeScript, gözüm üzerinde).
* Yeni bir dil oluşturmak sizi çalışma zamanı ortamlarınızdan ve topluluklarınızdan uzaklaşmanızı sağlar (Dart, gözüm üzerinde).

TypeScript, dökümanlarla JavaScript'tir.

## JavaScript'i İyileştirmek

TypeScript sizi hiç çalışmamış JavaScript parçalarından korumaya çalışacaktır ( Bu sebeple bunları hatırlamanıza gerek yok):

```ts
[] + []; // JavaScript size "" sonucunu verecek (ki mantıklı olan), TypeScript ise hata

//
// JavaScript'te anlamsız olan diğer konular
// - çalışma zamanı hatası vermez (hata ayıklamayı zorlaştırıyor)
// - ama TypeScript derleme zamanı hatası verecek (hata ayıklamayı gereksizleştiriyor)
//
{} + []; // JS : 0, TS Hata
[] + {}; // JS : "[object Object]", TS Hata
{} + {}; // JS : NaN, TS Hata
"merhaba" - 1; // JS : NaN, TS Hata

function ekle(a,b) {
  return
    a + b; // JS : undefined, TS Hata 'unreachable code detected'
}
```

Esasen TypeScript JavaScript'i potansiyel hatalar için analiz eder. Sadece benzer işi yapıp, tür bilgisi olmayan diğerlerinden daha iyi bir iş çıkarıyor.

## Hala JavaScript öğrenmeniz gerekiyor

Bunun anlamı TypeScript *JavaScript yazmanız* gerçeği hakkında oldukça faydalıdır. Yani gafil avlanmamanız için JavaScript ile ilgili hala farkına varmanız gereken bazı şeyler var. Gelin şimdi bunları tartışalım.