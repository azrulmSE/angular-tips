# angular-tips
Place where I put some of my angular tips, for my future me.

## Directives

### Symbols in scope object

1. "@"   (  Text binding / one-way binding )
2. "="   ( Direct model binding / two-way binding )
3. "&"   ( Behaviour binding / Method binding  )

```js
scope: {
        name: "@",
        color: "=",
        reverse: "&"
    }
```

```html
<div my-directive
  name="{{name}}"
  reverse="reverseName()"
  color="color" >
</div>
```


### Two-way data binding on directives

```html
  <my-directive value-binded="vm.variable"/>
```

```javascript

app.module('whatever').directive('my-directive', function() {
        return {
            restrict: 'E',
            scope: {
                valueBinded: '=valueBinded' // THIS IS THE IMPORTANT PART
            },
            controller: ['$scope', function($scope) {
              // binded value is accesible through scope
              console.log($scope.valueBinded) // Logs 
            }],
            template: '<input type="text" ng-model="valueBinded"/>';
        };
    });
    
```

### Activate directive controller when two-way data binding is loaded

Sometimes we need to wait until data is available so we can operate inside directive. Just add a $watch that run once, and when valid data is available run the activate() method.

Useful when we can show the template but running the controller would raise errors when running some methods working with value binded.

```javascript

app.module('whatever').directive('my-directive', function() {
        return {
            restrict: 'E',
            scope: {
                valueBinded: '=valueBinded' 
            },
            controller: ['$scope', function($scope) {
              // binded value is accesible through scope
              var unbindActivateListener = $scope.$watch("valueBinded", function(newVal, oldVal) {
                    if (typeof newVal !== "undefined") {  // Check if valueBinded contains valid information
                        unbindActivateListener();
                        activate();
                    }
                });
                
              function activate() {
                console.log($scope.valueBinded) // Logs 
              }
              
            }],
            template: '<input type="text" ng-model="valueBinded"/>';
        };
    });
    
```

### Link vs Controller, inside a directive

Link for only simple DOM manipulation and event handling.
Controller when logic or templates are expected to be _not that simple_.

## Testing

### Mocking a factory and injecting it in a controllerAs 

```javascript

describe('Controller: myCtrl as vm', function() {
  var myCtrl, scope;  // Declare here what you will use across the $provide and the tests

  // Initialize the controller and scope
  beforeEach(function() {
    // Load the controller's module
    module('myCtrlModule');

    // Provide any mocks needed
    module(function($provide) {
      // Provide all the dependencies injected on controller
      /*
      $provide.value('dependency1', {});
      */

      // Provide myFactory mock
      $provide.factory('myFactory', function($q) {  // Add $q as we will mock a promise response
        return {
          factoryMethod: jasmine.createSpy('factoryMethod').andCallFake(function(num) {
            return $q.when({
              "result": "this must be the result of your promise"
            });
          })
        };
      });
      
      // Add the rest of dependencies
      $provide.value('dependencyN', {});
    });


    inject(function($controller, _dependency1_, _myFactory_, _dependencyN_) {
      scope = {};
      
      // ADD dependencies to the controller
      myCtrl = $controller('myCtrl as vm', {
        $scope: scope,
        dependency1: _dependency1_,
        myFactory: _myFactory_,
        dependencyN: _dependencyN_
      });
    });

  });

  it('should exist', function() {
    expect(!!myCtrl).toBe(true);
  });

});

```
