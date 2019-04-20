# Ticker
Почему ```this._i``` не увеличивается. Как исправить?

```js
function Ticker() {
    this._i = 0
};
Ticker.prototype = {
     tick: function() {
     console.log(this._i++);
 }
};
var ticker = new Ticker();
setInterval(ticker.tick, 1000);
```

### Проблема:
Таймер не увеличиваеся так как в функцию ```setInterval``` в качестве первого аргумента передается функция которая при вызове использует обращение к контексту ```this```. В таком случае ```this``` при срабатываении таймера будет обьект глобального контекста исполнения(```window || global```), в результате ```this._i``` - ```undefiend```

### Решение:

##### Способ 1.0:
Самое очевидное решение это обернуть функцию которую мы передаем в ```setItnerval``` в анонимную функцию и вызвать её внутри: 

```js
function Ticker() {
    this._i = 0
};
Ticker.prototype = {
     tick: function() {
     console.log(this._i++);
 }
};
var ticker = new Ticker();
setInterval(function() {
    ticker.tick();
}, 1000);
```
Таким образом котнтекст ```this``` будет сохраняться так как функция использующая этот контекст не будет методом глобального обьекта, а только её функция-обертка и таймер будет работать

##### Способ 1.1:
Можно также использовать функицю для изменения контекста ```bind()``` которая создаст копию нашей функции с указаным контекстом который мы передали первым аргументом: 
```js
function Ticker() {
    this._i = 0
};
Ticker.prototype = {
     tick: function() {
     console.log(this._i++);
 }
};
var ticker = new Ticker();
setInterval(ticker.tick.bind(ticker), 1000);
```

##### Способ 1.2(Сомнительный):
в ```setInterval``` в качестве первого аргумента можно передать код обернутый в строчку, он будет выполняться каждый раз по истечению таймера:

```js
function Ticker() {
    this._i = 0
};
Ticker.prototype = {
     tick: function() {
     console.log(this._i++);
 }
};
var ticker = new Ticker();
setInterval('ticker.tick()', 1000);
```
##### Способ 2:
Источник: [MDN](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)

Можно переопределить ```setInterval``` на функцию которая будет проверять первый аргумент на тип ```Function``` и если это не функция то это будет контекст в котором эта функция будет вызвана: 

```js
  var __nativeSI__ = window.setInterval;
  window.setInterval = function (vCallback, nDelay ) {
    var oThis = this, aArgs = Array.prototype.slice.call(arguments, 2);
    return __nativeSI__(vCallback instanceof Function ? function () {
      vCallback.apply(oThis, aArgs);
    } : vCallback, nDelay);
  };
  function Ticker() {
    this._i = 0
  };
  Ticker.prototype = {
   tick: function() {
   console.log(this._i++);
   }
  };
  var ticker = new Ticker();
  setInterval.call(ticker,
    ticker.tick
  , 1000);
```
