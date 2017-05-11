## Hangi dosyalar?

Açık olarak ifade edebilmek için iki şekilde `files` kullanabilirsiniz:

```json
{
    "files":[
        "./some/file.ts"
    ]
}
```

veya `include` ve `exclude` ile dosyaları belirtebilirsiniz. Örneğin,


```json
{
    "include":[
        "./folder"
    ],
    "exclude":[
        "./folder/**/*.spec.ts",
        "./folder/someSubFolder"
    ]
}
```

Birkaç not:

* eğer `files` belirtildiyse diğer seçenekler yok sayılır
* `**/*` (örnek kullanımı `somefolder/**/*`) bütün klasörler ve bazı dosyalar anlamına gelir (`.ts`/`.tsx` dosya uzantıları dahil olacaktır ve eğer `allowJs` true ise `.js`/`.jsx` uzantıları bile bu uzantılara dahil olacaktır)
