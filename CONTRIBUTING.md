 # Katkıda bulunmak isteyenler için bilgilendirme:

*typescript-book kitabının yerelleştirmesini yaparken hedefimiz birebir çeviriden ziyade, bir Türk okuyucu tarafından bütünüyle anlaşılır bir kaynak oluşturmaktır.*

### Yerelleştirme İşlemi Aşamaları
1. Katkıda bulunmak isteyenler, ilgili repository'de açılan issue'lar içerisinden (https://github.com/CodeFiction/typescript-book/issues) bir görevi üstlenip, değişiklikleri ve çevirileri kendi kopyası (forku) üzerinden gerçekleştirir.
2. Gerekli değişiklikler yapıldıktan sonra ana repository'e Pull Request gönderilir.
	..* Ana reposory'e gelen pull request'ini **iki farklı kişi** gözden geçirir.
	..* İki ayrı kullanıcının da onayını alan pull request sonrasında ana repository'e merge (rebase and merge yöntemiyle) edilir.
	..* Gelen pull request'i merge işlemini gerçekleştiren kişi, aynı isimle sonuna "gözden geçirilecek" ekleyerek yeni bir issue açar. *(Açılan bu issue ile amacımız, yapılan değişikliklerin Türkçe dil ve imla kurallarına uygunluğunun denetlenmesini sağlamak.)*

### Ek Notlar
- Katkıda bulunmak istediğiniz konuya yorum bırakarak son durumu sorabilirsiniz. Katılmak için talepte bulunabilirsiniz.
- Atanmış bir issue ile ilgili yardımcı olabilecekseniz issue sahibi ile iletişime geçebilirsiniz.
- Üzerinizdeki görevi(issue) bırakmanız gerekirse lütfen topluluğa ilgili Gitter kanalı üzerinden haber veriniz.
- Teknik terimlerin çevirisinde bir bütünlük sağlamak amacı ile bir [sözlük](GLOSSARY.md) oluşturulmaktadır. Lütfen çevirinizi yaparken ve bir pull request'i incelerken bu sözlükte yer alan terimleri takip ediniz.
- Diğer görüş ve önerileriniz için gitter kanalına bekleriz. https://gitter.im/codefiction/typescript-book

### Kodları İndirirken
- Öncelikle [codefiction/typescript-book](https://github.com/CodeFiction/typescript-book) repository'sini **fork** yaparak kendinize bir kopyasını oluşturun.
- Daha sonra `git clone` komutunu kullanarak kendi bilgisayarınıza kendinize ait olan kopyayı indirin.
- Kopyayı indirdikten sonra `git remote` komutunu kullanarak **codefiction** repository'sini yeni bir *remote* olarak ekleyin.
```sh
git remote add upstream https://github.com/CodeFiction/typescript-book.git
```

#### Değişikliklerinizi Gönderirken
- Değişikliklerinizi yaptıktan sonra uygun **commit mesajı** ile değişikliklerinizi *commit* yapın.
```sh
git commit -m "Contributing kurallarına git mesajlarını ekliyorum"
```
- Yaptığınız değişikliği *push* yapmadan önce **git pull** komutunu kullanarak *upstream* i lokalinize indirin.
```sh
git pull upstream master --rebase
```
- Eğer merge konusunda bir sorun yaşamazsanız **git push** yapabilirsiniz.
```sh
git push origin master
```
- Artık GitHub üzerinden [**pull request**](https://help.github.com/articles/creating-a-pull-request/) oluşturabilirsiniz.

#### Pull Request Yorumlarını Düzeltirken
- Değişikliklerinizi yaptıktan sonra uygun **commit mesajı** ile değişikliklerinizi *commit* yapın.
```sh
git commit -m "Contributing kurallarını yorumlardan sonra düzelttim"
```
- Yaptığınız değişiklik *push* yapmadan önce **git pull** komutunu kullanarak *upstream* i lokalinize indirin.
```sh
git pull upstream master --rebase
```
- Eğer merge konusunda bir sorun yaşamazsanız **git push** yapabilirsiniz.
```sh
git push origin master
```
- Yeni bir *pull request* yapmanıza gerek kalmadan değişikliklerinizin önceki *pull request* içinde oluştuğunu göreceksiniz.

:tr: Codefiction Ekibi
