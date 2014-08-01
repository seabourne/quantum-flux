quantum-flux
====

A clean and understandable API that provides async, integrated, stores and
dispatchers for Facebook's React Framework(tm).

####Install:
```
  npm install quantum-flux
```

[Full API Documentation is here](http://sterpe.github.io/quantum-flux/out/Store.html)  It's a work in progress so please excuse the mess, but you will find full documentation for the `quantum.FluxInstance` and `FluxInstance.Store` classes. 

####Create a new flux instance for your application:

```javascript

  var quantum = require('quantum-flux');
  
  var flux = new quantum.FluxInstance();
  
  //dispatch a payload, it can be anything, though you'll probably prefer to use objects.

  flux.dispatch('hello, quantum flux!');

```

####Create a new store:

Any properties attached to the config object passed into the flux.Store constructor are attached to `this` store.
  
```javascript
  //If you just need a quick store...
  var aStore = new flux.Store();
  
  //Probably you'll eventually need to do something more complex,
  //the flux.Store constructor will extend the store with any properties passed it via a `conf` object.
  
  //We define the property `foo`, `funcB`, and `funcA` for `myStore` instance.
  var myStore = new flux.Store({
    foo: "bar",
    funcB: function () {/*...*/},
    funcA: function (payload) {
      //do something with the payload!
      
      //Store objects share the EventEmitter prototype -- they can emit events and attach listeners!
      this.emit('change', /*args to pass to this event*/);
    }
  });
```
####Registering and unregistering a store:

Now register this store and a listener function to the flux dispatcher!
```javascript

  var myStore = new flux.Store();
  
  //Note: the registered function will be called with `myStore` as the `this` value.
  myStore.register(function (payload) {
    this.emit('change');
  });
  
  //You can change or swap the registered function at any time and it takes effect immediately!
  myStore.register(myStore.funcA);
  
  //you can also unregister a store, the dispatcher will no longer send dispatches to this store.
  myStore.unregister();
```

####Stores are EventEmitters:

```javascript

  myStore.addListener('change', function listener(e /*, ...args */) {
    
    //have your View-controller do something in response to the event.
    
    myStore.removeListener(listener);  //You probably don't need to do this, but you can...
    
  });
  
  
```

####Waiting for other Stores:

Stores can wait for another store or stores.

```javascript

var storeA = new flux.Store(),
  storeB = new flux.Store();
  
  storeA.register(function (payload) {
    
    switch (payload.someCondition) {
    
      case foo:
        this.waitFor(storeB /* or an [ ] of stores! */, function onFulfilled (payload) {
        
          //do something now that storeB is done
        
          this.emit('change');
        }, function onError (e) {
        
          //do whatever in case there was an error in storeB
        });
        break;
        
      default:
        return;
    }
    
  });
  
```

####All emits that occured during a dispatch phase are handled, in registration order, synchronously.

When a store calls `this.emit(e, [args])`, that call is tied into to the dispatch phase/cycle...sort of like the way
machine guns on WWI aircraft were linked to the engine cycle, to avoid shooting off the propellor.  This is also similar to the way that AngularJS's $setTimeout function is tied into the Angular digest cycle, for those familiar with Angular.

`emit` calls are queued and fired in order synchronously, so we never have to worry about a browser redraw occuring between the point where all the stores have completed processing the dispatch and all listeners of those stores have executed their callbacks.

There shouldn't be dependency on side effects between listeners of separate stores for the same event.  If two stores take action in response to the same dispatch, the listeners for each store are fired in store registration order.  This provides for reasonable consistency of callback order.

In the example above, if both storeA & storeB emit events to their listeners, storeA's eventListeners will be called,
before storeB's, even though storeA waited on storeB.


####Finally, if you really want to get crazy, you can extend from the Store.prototype:

```javascript

function myCustomComplicatedStore {

//Yadda, yadda, yadda...

}

favoriteExtendFunc(myCustomComplicatedStore.prototype, flux.Store.prototype);

//Wha-la.
```

