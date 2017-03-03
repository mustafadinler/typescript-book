# Genel Hatalar
Bu bölümde kullanıcıların gerçek hayatta deneyimledikleri birkaç genel hata kodunu açıklayacağız.

## TS2304
Örnek:
> `Cannot find name ga (ga ismi bulunamıyor)`

Muhtemelen `declare (deklare)` etmediğiniz üçüncü parti bir kütüphane (örneğin google analytics) kullanıyorsunuz. TypeScript sizi *yazım hatalarından* ve *değişkenleri deklare etmeden kullanmanızdan* korumaya çalışır, yani *çalışma zamanında varolan* harhangi bir şeye açık olmalısınız çünkü bazı dış kütüphaneleri dahil ediyorsunuz ([nasıl düzeltileceğine dair dahası][ambient]).

## TS2307
Örnek:
> `Cannot find module 'underscore' ('underscrore' modülü bulunamıyor)`

Muhtemelen *modül* ([modüllere (modules) dair dahası][modules]) olarak üçüncü parti bir kütüphane (örneğin underscore) kullanıyorsunuz ve bunun için içsel deklarasyon dosyanız yok ([içsel deklarasyona dair dahası][ambient]).

## TS1148
Örnek:
> Cannot compile modules unless the '--module' flag is provided ('--module' işareti sağlanmadığı sürece modüller derlenemez)

[Modüller (modules) bölümü][modules]'ne göz atın.

## Arama indekslemesi için
Bu kısmı görmezden gelebilirsiniz. Bu kısım arama motoru indekslemesi için.

> Kullanıcıların kullanmayı sevdiği ve hata vermeye eğilimli diğer modüller:
* Cannot find name $ ($ ismi bulunamıyor)
* Cannot find module jquery (jquery modülü bulunamıyor)

[ambient]: ../types/ambient/d.ts.md
[modules]: ../project/modules.md
