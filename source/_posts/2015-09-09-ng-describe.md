title: How to improve your Angular unit tests with ng-describe
date: 2015/09/09 01:00
authorId: BMU
tags: [angular, testing]
---

## Improve your Angular.js unit tests with ng-describe

### Overview
I had a some time this weekend and was exploring the ways we could improve our javascript unit tests. The one thing I do not really like about angular unit testing is too much boilerplate code. Probably you all know something like this:

```javascript
beforeEach(module('blabla'));
beforeEach( inject ( function ($controller, $rootScope, _$q_) {
        $q = _$q_;
        $scope = $rootScope.$new();
        controller = $controller('createUserController', {
            $scope: $scope,
            TeamsService: TeamsService,
            CountriesService: CountriesService,
            Toggles: Toggles
        });
 }));
```

It's not very exciting. Sure, you could create some level of abstraction above it and reuse it in all your tests, but it’s not so trivial and probably need significant amount of time to implement.

And here is a little utility library [ng-describe](https://github.com/kensho/ng-describe) to help. 

<!-- more -->

### Setup
* `npm i ng-describe -D` or `bower i ng-describe -D`
* put it in Karma configuration after angular.js and angular-mocks.js, but before your test files.

### Examples
Here are some examples of our tests that was rewritten with ng-describe.
> Disclaimer: there is no sense to rewrite all your existing tests. I did only few, just to evaluate if it's suitable for our future tests.

Before:

```javascript
'use strict';

describe('prOppsControllerTest', function () {

    beforeEach(module(‘mojo’));

    var $scope;
    var proppsController;
    var mockPrOpportunitiesService;
    var mockSectorsService;
    var $q;
    var deferredPrOppSearch;
    var deferredSectorsSearch;

    beforeEach(inject(function ($controller, $rootScope, _$q_) {
        $scope = $rootScope.$new();
        $q = _$q_;

        mockPrOpportunitiesService = {
            search: function() {
                deferredPrOppSearch = $q.defer();
                return deferredPrOppSearch.promise;
            }
        };

        mockSectorsService = {
            search: function() {
                deferredSectorsSearch = $q.defer();
                return deferredSectorsSearch.promise;
            }
        };

        $scope.session = {};
        $scope.session.user = { country: { id: 32 } };

        proppsController = $controller('proppsController', {
            $scope: $scope,
            PrOpportunitiesService: mockPrOpportunitiesService,
            SectorsService: mockSectorsService
        });
    }));

    it('should search propps', function(){
        $scope.search();

        deferredSectorsSearch.resolve({id: 2});
        deferredPrOppSearch.resolve([{id: 1}, {id: 2}]);

        $scope.$root.$digest();

        expect($scope.loadingPropps).toBeFalsy();
        expect($scope.prOpps).not.toBeNull();
        expect($scope.prOpps.length).toBe(2);
    });

});
```

After:

```javascript
'use strict';

var deferredPrOppSearch;
var deferredSectorsSearch;

ngDescribe({
    name: 'prOppsControllerTest',
    modules: 'mojo',
    controllers: 'proppsController',
    inject: ['PrOpportunitiesService'],
    mocks: {
        mojo: {
            PrOpportunitiesService: {
                search: function($q) {
                    deferredPrOppSearch = $q.defer();
                    return deferredPrOppSearch.promise;
                }
            },
            SectorsService: {
                search: function($q) {
                    deferredSectorsSearch = $q.defer();
                    return deferredSectorsSearch.promise;
                }
            }
        }
    },
    tests: function(deps) {
        beforeEach(function() {
            deps.$rootScope.session = {};
            deps.$rootScope.session.user = { country: { id: 32 } };
        });

        it('should search propps', function() {
            var scope = deps.proppsController;

            spyOn(deps.PrOpportunitiesService, 'search').andCallThrough();

            scope.search();

            deferredPrOppSearch.resolve([{id: 1}, {id: 2}]);

            scope.$apply();

            expect(scope.loadingPropps).toBeFalsy();
            expect(scope.prOpps).not.toBeNull();
            expect(scope.prOpps.length).toBe(2);

            expect(deps.PrOpportunitiesService.search).toHaveBeenCalled();
        });
    }
});
```

One more example of service test.

Before:

```javascript
'use strict';

describe('Frequency Service', function() {
    var frequencyService;
    var httpBackend;

    beforeEach(module('mojo'));

    beforeEach(inject(
        ['FrequencyService', '$httpBackend', function(service, $httpBackend) {
        frequencyService = service;
        httpBackend = $httpBackend;
    }]));

    afterEach(function(){
        httpBackend.flush();
    });

    it('should GET all frequencies and return in promise', function() {
        httpBackend.expectGET('/frequencies').respond(200, [{id: 1},{id: 2}]);

        var promise = frequencyService.get();

        promise.then(function(data){
            expect(data.length).toBe(2);
        });
    });
});
```

After:

```javascript
'use strict';

ngDescribe({
    name: 'Frequency Service',
    modules: 'mojo',
    inject: ['FrequencyService', '$httpBackend'],
    tests: function(deps) {
        afterEach(function() {
            deps.$httpBackend.flush();
            deps.$httpBackend.verifyNoOutstandingExpectation();
            deps.$httpBackend.verifyNoOutstandingRequest();
        });

        it('should GET all frequencies and return in promise', function() {
            deps.$httpBackend.expectGET('/frequencies').respond(200, [{id: 1}, {id: 2}]);
            deps.FrequencyService.get().then(function(result) {
                expect(result.length).toBe(2);
            });
        });
    }
});
```

And finally directive test.

Before:
```javascript
describe('open-url directive', function() {

    var scope;
    var compile;
    var window;

    beforeEach(module('mojo'));
    beforeEach(inject(function($compile, $rootScope, $window) {
        compile = $compile;
        scope = $rootScope;
        $window.open = function(url) {}; // mock
        window = $window;
    }));

    it('should correctly open window when URL w/ optional protocol', function() {
        var element = angular.element('<div open-url="url">');
        element = compile(element)(scope);
        scope.url = 'www.google.com';
        scope.$digest();

        spyOn(window, 'open');
        element.click();
        expect(window.open).toHaveBeenCalledWith('http://www.google.com');
    });

});
```

After:

```javascript
'use strict';

ngDescribe({
    name: 'open-url directive',
    modules: 'mojo',
    inject: ['$window'],
    element: '<div open-url="url">',
    tests: function(deps) {
        it('should correctly open window when URL w/ optional protocol', function() {
            var scope = deps.element.scope();
            scope.url = 'www.google.com';

            spyOn(deps.$window, 'open');

            scope.$apply();

            deps.element.click();
            expect(deps.$window.open).toHaveBeenCalledWith('http://www.google.com');
        });
    }
});
```

### Summary:
I think ng-describe provides us with more clean and transparent way to write our unit tests. It is clear what you are trying to test, to mock or inject. And as a bonus with fewer lines of code.