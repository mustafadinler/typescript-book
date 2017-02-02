### Yineleyiciler

Yineleyici kavramı ne TypeScript ne de ES6'e ait bir özelliktir. Yineleyici Nesneye Dayalı Programlama dilleri için ortak bir davranışsal tasarım desenidir.
Bu kavram, genellikle, aşağıdaki arayüzü uygulayan nesnedir:

```ts
interface Iterator<T> {
    next(value?: any): IteratorResult<T>;
    return?(value?: any): IteratorResult<T>;
    throw?(e?: any): IteratorResult<T>;
}
```

Bu arayüz, nesneye ait olan bir koleksiyon veya diziden değer elde etmeyi sağlar.

Kendisini oluşturan bileşenlerin listesini içeren yapıya ait bir nesne tasarlanmış olduğunu düşünün.
Yineleyici arayüzü ile bu nesneden bileşenlerini almak aşağıdaki gibi mümkündür:

```ts
'use strict';

class Component {
  constructor (public name: string) {}
}

class Frame implements Iterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true
      }
    }
  }

}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
let iteratorResult1 = frame.next(); //{ done: false, value: Component { name: 'top' } }
let iteratorResult2 = frame.next(); //{ done: false, value: Component { name: 'bottom' } }
let iteratorResult3 = frame.next(); //{ done: false, value: Component { name: 'left' } }
let iteratorResult4 = frame.next(); //{ done: false, value: Component { name: 'right' } }
let iteratorResult5 = frame.next(); //{ done: true }

//Yineleyici sonucunun değerine 'value' özelliği aracılığıyla ulaşmak mümkündür.
let component = iteratorResult1.value; //Component { name: 'top' }
```

Tekrar belirtmek gerekirse yineleyici bir TypeScript özelliği değildir. Bu kod Iterator ve IteratorResult arayüzleri kullanılmadan da çalışabilir.

Ancak, kod tutarlılığı için, bu ortak ES6 arayüzlerini (./types/interfaces.md) kullanmak çok yararlı olacaktır.

Peki, iyi guzel de, daha da yararlı nasıl kullanılabilir? ES6 [Symbol.iterator] `sembol` içeren *tekrarlanabilir protokol* tanımlar. 
Iterable arayüzü şu şekilde uygulanabilir:

```ts
//...
class Frame implements Iterable<Component> {

  constructor(public name: string, public components: Component[]) {}

  [Symbol.iterator]() {

    let pointer = 0;
    let components = this.components;

    return {

      next(): IteratorResult<Component> {
        if (pointer < components.length) {
          return {
            done: false,
            value: components[pointer++]
          }
        } else {
          return {
            done: true
          }
        }
      }

    }

  }

}

let frame = new Frame("Door", [new Component("top"), new Component("bottom"), new Component("left"), new Component("right")]);
for (let cmp of frame) {
  console.log(cmp);
}
```

Ne yazık ki bu tasarımla `frame.next()` çalışmaz ve bir takım işler eksikmiş gibi görünür. 
IterableIterator arayüzü durumu kurtarmak için imdadımıza şu şekilde yetişir:

```ts
//...
class Frame implements IterableIterator<Component> {

  private pointer = 0;

  constructor(public name: string, public components: Component[]) {}

  public next(): IteratorResult<Component> {
    if (this.pointer < this.components.length) {
      return {
        done: false,
        value: this.components[this.pointer++]
      }
    } else {
      return {
        done: true
      }
    }
  }

  [Symbol.iterator](): IterableIterator<Component> {
    return this;
  }

}
//...
```

IterableIterator arayüzü ile `frame.next()` ve `for` döngüleri olması gerektiği gibi çalışabilir.

Yineleyicinin sonlu bir değeri yinelemesi gerekmez.
Bununla ilgili tipik örnek olarak Fibonacci dizisi şu şekilde uygulanır:

```ts
class Fib implements IterableIterator<number> {

  protected fn1 = 0;
  protected fn2 = 1;

  constructor(protected maxValue?: number) {}

  public next(): IteratorResult<number> {
    var current = this.fn1;
    this.fn1 = this.fn2;
    this.fn2 = current + this.fn1;
    if (this.maxValue && current <= this.maxValue) {
      return {
        done: false,
        value: current
      }
    } return {
      done: true
    }

  }

  [Symbol.iterator](): IterableIterator<number> {
    return this;
  }

}

let fib = new Fib();

fib.next() //{ done: false, value: 0 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 1 }
fib.next() //{ done: false, value: 2 }
fib.next() //{ done: false, value: 3 }
fib.next() //{ done: false, value: 5 }

let fibMax50 = new Fib(50);
console.log(Array.from(fibMax50)); // [ 0, 1, 1, 2, 3, 5, 8, 13, 21, 34 ]

let fibMax21 = new Fib(21);
for(let num of fibMax21) {
  console.log(num); //Prints fibonacci sequence 0 to 21
}
```

#### ES5 hedef alınarak yineleyicilerle kod oluşturma
Yukarıdaki kod örneklerinin çalıştırılabilmesi için ES6 hedef olarak gösterilmesi gerekmektedir, ancak ES5 kullanarak, `Symbol.iterator` yapısını destekleyen bir JS motoru ile de kodlar derlenebilir.
Buda ES5 hedef alınarak, ES6 kütüphanesi kullanılarak gerçekleştirilebilir(Projenize es6.d.ts kütüphanesini ekleyip derleyin.)
Derlenmiş kod node 4+, Google Chrome ve bazı diğer tarayıcılarda çalışacaktır.

