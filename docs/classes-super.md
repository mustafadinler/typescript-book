#### `super`

Dikkat edin eğer alt sınıfınızda `super`çağrırsanız bu sizi aşağıda gösterildiği gibi `prototype`'a yönlendirecektir:

```ts
class Base {
    log() { console.log('merha dünya'); }
}

class Child extends Base {
    log() { super.log() };
}
```
şunu üretir:

```js
var Base = (function () {
    function Base() {
    }
    Base.prototype.log = function () { console.log('merha dünya'); };
    return Base;
})();
var Child = (function (_super) {
    __extends(Child, _super);
    function Child() {
        _super.apply(this, arguments);
    }
    Child.prototype.log = function () { _super.prototype.log.call(this); };
    return Child;
})(Base);

```
`_super.prototype.log.call(this)`'e dikkat.

Bunun anlamı eleman özelliklerinizde `super`'i kullanamazsınız. Bunun yerine sadece `this` kullanmalısınız.

```ts
class Base {
    log = () => { console.log('merhaba dünya'); }
}

class Child extends Base {
    logWorld() { this.log() };
}
```

`Base` ve `Child` sınıf arasında paylaşılan tek bir tane `this` olduğundan *farklı* adları kullanmanız gerektiğine dikkat edin (işte `log` ve `logWorld`).

Ayrıca, `super`'i yanlış kullanmaya çalışırsanız TypeScript sizi uyaracaktır:

```ts
module quz {
    class Base {
        log = () => { console.log('merhaba dünya'); }
    }

    class Child extends Base {
        // ERROR : only `public` and `protected` methods of base class are accessible via `super`
        // HATA : sadece üst sınıfın `public` ve `protected` metotları `super` ile erişilebilirdir
        logWorld() { super.log() };
    }
}
```
