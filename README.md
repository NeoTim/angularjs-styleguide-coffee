# AngularJS Style Guide

*Opinionated AngularJS Coffeescript style guide for teams by [@JoelCox](//twitter.com/joelcoxokc)*

Original Javascript style guide by [@john_papa](//twitter.com/john_papa)

If you are looking for an opinionated style guide for syntax, conventions, and structuring AngularJS applications, then step right in. The styles contained here are based on on my experience with [AngularJS](//angularjs.org), presentations, [Pluralsight training courses] (http://pluralsight.com/training/Authors/Details/john-papa) and working in teams.

The purpose of this style guide is to provide guidance on building AngularJS applications by showing the conventions I use and, more importantly, why I choose them.

## Community Awesomeness and Credit
Never work in a vacuum. I find that the AngularJS community is an incredible group who are passionate about sharing experiences. As such, a friend  and  AngularJS expert Todd Motto and I have collaborated on many styles and conventions. We agree on most, and some we diverge. I encourage you to check out [Todd's  guidelines](https://github.com/toddmotto/angularjs-styleguide) to get a sense for his approach and how it compares.

Many of my styles have been from the many pair programming sessions [Ward Bell](http://twitter.com/wardbell) and I have had. While we don't always agree, my friend Ward's has certainly helped influence the ultimate evolution of this guide.



## Table of Contents

  1. [Single Responsibility](#single-responsibility)
  1. [Modules](#modules)
  1. [Controllers](#controllers)
  1. [Services](#services)
  1. [Factories](#factories)
  1. [Directives](#directives)
  1. [Resolving Promises for a Controller](#resolving-promises-for-a-controller)
  1. [Manual Dependency Injection](#manual-dependency-injection)
  1. [Minification and Annotation](#minification-and-annotation)
  1. [Exception Handling](#exception-handling)
  1. [Naming](#naming)
  1. [Application Structure](#application-structure)
  1. [Modularity](#modularity)
  1. [Angular $ Wrapper Services](#angular-$-wrapper-services)
  1. [Comments](#comments)
  1. [JSHint](#js-hint)
  1. [Angular Docs](#angular-docs)
  1. [Contributing](#contributing)
  1. [License](#license)

## Single Responsibility

  - **Rule of 1**: Define 1 component per file.

 	The following example defines the `app` module and its dependencies, defines a controller, and defines a factory all in the same file.

    ```coffeescript
    # avoid

    SomeController = ()->

    someFactory = ()->

    angular
    	.module('app', ['ngRoute'])
    	.controller('SomeController' , SomeController)
    	.factory('someFactory' , someFactory)

    ```

	The same components are now separated into their own files.

    ```coffeescript
    # recommended

    # app.module.js
    angular
    	.module('app', ['ngRoute'])
    ```

    ```coffeescript
    ### recommended ###

    # someController.js

    SomeController = ()->

    angular
    	.module('app')
    	.controller('SomeController' , SomeController)

    ```

    ```coffeescript
    ### recommended ###

    # someFactory.js
    someFactory = ()->

    angular
    	.module('app')
    	.factory('someFactory' , someFactory)

    ```

**[Back to top](#table-of-contents)**

## Modules

  - **Definitions (aka Setters)**: Declare modules without a variable using the setter syntax.

	*Why?*: With 1 component per file, there is rarely a need to introduce a variable for the module.

    ```coffeescript
    ### avoid ###
    app = angular.module('app', [
        'ngAnimate'
        'ngRoute'
        'app.shared'
        'app.dashboard'
    ])
    ```

	Instead use the simple getter syntax.

    ```coffeescript
    ### recommended ###
    angular
    	.module('app', [
        'ngAnimate'
        'ngRoute'
        'app.shared'
        'app.dashboard'
    ])
    ```

  - **Getters**: When using a module, avoid using a variables and instead use   chaining with the getter syntax.

	*Why?* : This produces more readable code and avoids variables collisions or leaks.

    ```coffeescript
    ### avoid ###
    app = angular.module('app')
    app.controller('SomeController' , SomeController)

    SomeController = ()->

    ```

    ```coffeescript
    ### recommended ###
    SomeController = ()->

    angular
      .module('app')
      .controller('SomeController' , SomeController)
    ```

  - **Setting vs Getting**: Only set once and get for all other instances.

	*Why?*: A module should only be created once, then retrieved from that point and after.

  	  - Use `angular.module('app', [])` to set a module.
  	  - Use  `angular.module('app')` to get a module.

  - **Named vs Anonymous Functions**: Use named functions instead of passing an anonymous function in as a callback.

	*Why?*: This produces more readable code, is much easier to debug, and reduces the amount of nested callback code.

    ```coffeescript
    ### avoid ###
    angular
      .module('app')
      .controller('Dashboard', ()->)
      .factory('logger', ()-> )
    ```

    ```coffeescript
    ### recommended ###

    # dashboard.js
    Dashboard = ()->

      # logic goes here -->

      return

    angular
      .module('app')
      .controller('Dashboard', Dashboard)
    ```

    ```coffeescript
    # logger.js

    logger = ()->

      # logic goes here -->

      return

    angular
      .module('app')
      .factory('logger', logger)
    ```

  - **IIFE**: Wrap AngularJS components in an Immediately Invoked Function Expression (IIFE).

	*Why?*: An IIFE removes variables from the global scope. This helps prevent variables and function declarations from living longer than expected in the global scope, which also helps avoid variable collisions.

    ```coffeescript
    (->
      logger = ()->

        # logic goes here -->

        return

      angular
        .module('app')
        .factory('logger', logger);

    )()
    ```

  - Note: For brevity only, the rest of the examples in this guide may omit the IIFE syntax.

**[Back to top](#table-of-contents)**

## Controllers

  - **controllerAs View Syntax**: Use the [`controllerAs`](http://www.johnpapa.net/do-you-like-your-angular-controllers-with-or-without-sugar/) syntax over the `classic controller with $scope` syntax.

	*Why?*: Controllers are constructed, "newed" up, and provide a single new instance, and the `controllerAs` syntax is closer to that of a JavaScript constructor than the `classic $scope syntax`.

	*Why?*: It promotes the use of binding to a "dotted" object in the View (e.g. `customer.name` instead of `name`), which is more contextual, easier to read, and avoids any reference issues that may occur without "dotting".

	*Why?*: Helps avoid using `$parent` calls in Views with nested controllers.

    ```html
    <!-- avoid -->
    <div ng-controller="Customer">
      {{ name }}
    </div>
    ```

    ```html
    <!-- recommended -->
    <div ng-controller="Customer as customer">
      {{ customer.name }}
    </div>
    ```

  - **controllerAs Controller Syntax**: Use the `controllerAs` syntax over the `classic controller with $scope` syntax.

  - The `controllerAs` syntax uses `this` inside controllers which gets bound to `$scope`

	  *Why?*: `controllerAs` is syntactic sugar over `$scope`. You can still bind to the View and still access `$scope` methods.

	  *Why?*: Helps avoid the temptation of using `$scope` methods inside a controller when it may otherwise be better to avoid them or move them to a factory. Consider using `$scope` in a factory, or if in a controller just when needed. For example when publishing and subscribing events using [`$emit`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$emit), [`$broadcast`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$broadcast), or [`$on`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on) consider moving these uses to a factory and invoke form the controller.

	  *NOTE!*: Considering CoffeeScript automatically returns the last line, we must place a return statement at the bottom so the controller doesn't *return* anything. In most cases you do not need the return statement. However, I ran into a few issues not using it while running tests on these functions. So I highly recommend using it. 

    ```coffeescript
    ### avoid ###
    (->
      Customer = ($scope)->
        $scope.name = {}
        $scope.sendMessage = ()->
      angular
        .module('app')
        .controller('Customer', Customer)
    )()


    ### recommended - but see next section ###
    (->
      Customer = ()->
        @name = {}
        @sendMessage = ()->

        return

      angular
        .module('app')
        .controller('Customer', Customer)
    )()
    ```

   	  - **controllerAs with vm**: Use a capture variable for `this` when using the `controllerAs` syntax. Choose a consistent variable name such as `vm`, which stands for ViewModel.

    	  *Why?*: The `this` keyword is contextual and when used within a function inside a controller may change its context. Capturing the context of `this` avoids encountering this problem.

    ```coffeescript
    ### avoid ###
    (->
      Customer = ()->
        @name = {}
        @sendMessage = ()->

          # here @/this is not the same
          @stuff = "stuff"

        return
      angular
        .module('app')
        .controller('Customer', Customer)
    )()

    ```

    ```coffeescript
    ### recommended ###
    (->
      Customer = ()->
        vm = @
        vm.name = {}
        vm.sendMessage = ()->

        return
      angular
        .module('app')
        .controller('Customer', Customer)
    )()

    ### OR use the fat arrow in functions => ###
    (->
      Customer = ()->
        @name = {}
        @sendMessage = ()=>
          @stuff

        return
      angular
        .module('app')
        .controller('Customer', Customer)
    )()
    ```

  - Note: You can avoid any [jshint](http://www.jshint.com/) warnings by placing the comment below above the line of code.

  ```coffeescript
  ### jshint validthis: true ###
  vm = @
  ```

  - **Bindable Members Up Top**: Place bindable members at the top of the controller, alphabetized, and not spread through the controller code.

    *Why?*: Placing bindable members at the top makes it easy to read and helps you instantly identify which members of the controller can be bound and used in the View.

    *Why?*: Setting anonymous functions inline can be easy, but when those functions are more than 1 line of code they can reduce the readability.

    *NOTE*: The javascript version of this style guide uses hoisted functions. CoffeeScript does not provide the ability do use function declarations (hoisted functions).

  ```javascript
  // function declaration
  function someFunction() { }; 
  /* vs */
  var someFunction = function() { };
  ```
 Defining the functions below the bindable members (the functions will be hoisted) moves the implementation details down, keeps the bindable members up top, and makes it easier to read.

    *Considering*: CoffeeScript doesn't provide hoisted functions we have to wrap any bindable variables in a function. After the machine reads the entire controller, we call the function. In this example I used a function named "init".

    ```coffeescript
    ### avoid ###
    (->
      Sessions = ()->

        @gotoSession = ()=>
          ### ... ###

        @refresh = ()=>
          ### ... ###

        @search = ()=>
          ### ... ###

        @sessions = []
        @title = 'Sessions'
      angular
        .module('app')
        .controller('Sessions', Sessions)
    )()
    ```

    ```coffeescript
    ### recommended ###
    (->
      Sessions = ()->

        init = ()=>

          @gotoSession = gotoSession
          @refresh = refresh
          @search = search
          @sessions = []
          @title = 'Sessions'

        ###########

        gotoSession = ()=>
          # ... #

        refresh = ()=>
          # ... #

        search = ()=>
          # ... #

        init()

        return
      angular
        .module('app')
        .controller('Sessions', Sessions)
    )()

    ```

  - **Defer Controller Logic**: Defer logic in a controller by delegating to services and factories.

    *Why?*: Logic may be reused by multiple controllers when placed within a service and exposed via a function.

    *Why?*: Logic in a service can more easily be isolated in a unit test, while the calling logic in the controller can be easily mocked.

    *Why?*: Removes dependencies and hides implementations details from the controller.

    ```coffeescript

    ### avoid ###
    (->
      Order = ( $http, $q )->

        @checkCredit = checkCredit
        @total = 0

        checkCredit = ()=>
          orderTotal = @total
          $http.get('api/creditcheck').then (data)=>
              remaining = data.remaining
              return $q.when(!!(remaining > orderTotal))
      angular
        .module('app')
        .controller('Order', Order)
    )()
    ```

    ```coffeescript
    ### recommended ###
    (->
      Order = (creditService)->

        init = ()->

          @checkCredit = checkCredit
          @total = 0

        checkCredit = ()->

          creditService.check()

        init()

        return

      angular
        .module('app')
        .controller('Order', Order)
    )()
    ```

  - **Assigning Controllers**: When a controller must be paired with a view and either component may be re-used by other controllers or views, define controllers along with their routes.

    - Note: If a View is loaded via another means besides a route, then use the `ng-controller="Avengers as vm"` syntax.

    *Why?*: Pairing the controller in the route allows different routes to invoke different pairs of controllers and views. When controllers are assigned in the view using [`ng-controller`](https://docs.angularjs.org/api/ng/directive/ngController), that view is always associated with the same controller.

   ```coffeescript

    ### avoid - when using with a route and dynamic pairing is desired ###

    # route-config.js
    (->
      config = ($routeProvider)->
        $routeProvider
          .when('/avengers', {
            templateUrl: 'avengers.html'
          })

      angular
        .module('app')
        .config(config)
    )()

    ```

    ```html
    <!-- avengers.html -->
    <div ng-controller="Avengers as vm">
    </div>
    ```

    ```coffeescript
    ### recommended ###

    # route-config.js

    (->
      config = ($routeProvider)->
        $routeProvider
          .when('/avengers', {
            templateUrl: 'avengers.html'
            controller: 'Avengers as vm'

          });
      angular
        .module('app')
        .config(config)
    )()
    ```

    ```html
    <!-- avengers.html -->
    <div>
    </div>
    ```

**[Back to top](#table-of-contents)**

## Services

  - **Singletons**: Services are instantiated with the `new` keyword, use `this` for public methods and variables. Can also use a factory, which I recommend for consistency.

  - Note: [All AngularJS services are singletons](https://docs.angularjs.org/guide/services). This means that there is only one instance of a given service per injector.

    ```coffeescript
    # service
    (->
      logger = ()=>
        @logError = (msg)=>
          # ... #


      angular
          .module('app')
          .service('logger', logger)
    )()
    ```

    ```coffeescript
    ## factory
    (->
      logger = ()->

        return {
          logError: (msg) ->
            # ... #

        }

      angular
          .module('app')
          .factory('logger', logger)
    )()
    ```

**[Back to top](#table-of-contents)**

## Factories

  - **Single Responsibility**: Factories should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle), that is encapsulated by its context. Once a factory begins to exceed that singular purpose, a new factory should be created.

  - **Singletons**: Factories are singletons and return an object that contains the members of the service.

  - Note: [All AngularJS services are singletons](https://docs.angularjs.org/guide/services).

  - **Public Members Up Top**: Expose the callable members of the service (it's interface) at the top, using a technique derived from the [Revealing Module Pattern](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript).

    *Why?*: Placing the callable members at the top makes it easy to read and helps you instantly identify which members of the service can be called and must be unit tested (and/or mocked).

    *Why?*: This is especially helpful when the file gets longer as it helps avoid the need to scroll to see what is exposed.

    *Why?*: Setting functions as you go can be easy, but when those functions are more than 1 line of code they can reduce the readability and cause more scrolling. Defining the callable interface via the returned service moves the implementation details down, keeps the callable interface up top, and makes it easier to read.


    ```coffeescript
    ### avoid ###
    (->
      dataService = ()->

        someValue = ''

        save = ()->
          # ... #

        validate = ()->
          # ... #

        return
          save: save,
          someValue: someValue,
          validate: validate

      angular
        .module('app')
        .service('dataService', dataService)
    )()


    ```

    ```coffeescript
    ### recommended ###
    (->
      dataService = ()->

        someValue = ''

        ##########

        return
          save: ()->
           # . #

          validate: ()->
           # . #
      angular
        .module('app')
        .service('dataService', dataService)
    )()

    ```
  - This way bindings are mirrored across the host object, primitive values cannot update alone using the revealing module pattern

**[Back to top](#table-of-contents)**

## Directives
- **Limit 1 Per File**: Create one directive per file. Name the file for the directive.

    *Why?*: It is easy to mash all the directives in one file, but difficult to then break those out so some are shared across apps, some across modules, some just for one module. Also easier to maintain.

    ```coffeescript
    ### avoid ###
    angular
      .module('app.widgets')

      # order directive that is specific to the order module
      .directive('orderCalendarRange', orderCalendarRange)

      # sales directive that can be used anywhere across the sales app
      .directive('salesCustomerInfo', salesCustomerInfo)

      # spinner directive that can be used anywhere across apps
      .directive('sharedSpinner', sharedSpinner)

      ### implementation details ###
    ```

    ```coffeescript
    ### recommended ###

     ###
     # @desc order directive that is specific to the order module at a company named Acme
     # @file calendarRange.directive.js
     # @example <div acme-order-calendar-range></div>
     ###
    angular
      .module('sales.order')
      .directive('acmeOrderCalendarRange', orderCalendarRange)

     ###
     # @desc spinner directive that can be used anywhere across the sales app at a company named Acme
     # @file customerInfo.directive.js
     # @example <div acme-sales-customer-info></div>
     ###
    angular
      .module('sales.widgets')
      .directive('acmeSalesCustomerInfo', salesCustomerInfo)

     ###
     # @desc spinner directive that can be used anywhere across apps at a company named Acme
     # @file spinner.directive.js
     # @example <div acme-shared-spinner></div>
     ###
    angular
      .module('shared.widgets')
      .directive('acmeSharedSpinner', sharedSpinner)

      ### implementation details ###
    ```

    -Note: There are many naming options for directives, especially since they can be used in narrow or wide scopes. Choose one the makes the directive and it's file name distinct and clear. Some examples are below, but see the naming section for more recommendations.

- **Limit DOM Manipulation**: When manipulating the DOM directly, use a directive. If alternative ways can be used such as using CSS to set styles or the [animation services](https://docs.angularjs.org/api/ngAnimate), Angular templating, [`ngShow`](https://docs.angularjs.org/api/ng/directive/ngShow) or [`ngHide`](https://docs.angularjs.org/api/ng/directive/ngHide), then use those instead. For example, if the directive simply hide and shows, use ngHide/ngShow, but if the directive does more, combining hide and show inside a directive may improve performance as it reduces watchers.

    *Why?*: DOM manipulation can be difficult to test, debug, and there are often better ways (e.g. CSS, animations, templating)

- **Restrict to Elements and Attributes**: When creating a directive that makes sense as a standalone element, allow restrict `E` (custom element) and optionally restrict `A` (custom attribute). Generally, if it could be its own control, `E` is appropriate. General guideline is allow `EA` but lean towards implementing as an element when its standalone and as an attribute when it enhances its existing DOM element.

    *Why?*: It makes sense.

    *Why?*: While we can allow the directive to be used as a class, if the directive is truly acting as an element it makes more sense as an element or at least as an attribute.

    ```html
    <!-- avoid -->
    <div class="my-calendar-range"></div>
    ```

    ```coffeescript
    ### avoid ###
    (->
      myCalendarRange = ()->
          link = (scope, element, attrs)->
            # ... #

          directive =
            link: link,
            templateUrl: '/template/is/located/here.html',
            restrict: 'C'

          return directive

      angular
          .module('app.widgets')
          .directive('myCalendarRange', myCalendarRange)
    )()
    ```

    ```html
    <!-- recommended -->
    <my-calendar-range></my-calendar-range>
    <div my-calendar-range></div>
    ```

    ```coffeescript
    ### recommended ###
    (->

      myCalendarRange = ()->
            
          directive =
              templateUrl: '/template/is/located/here.html',
              restrict: 'EA',
              link: (scope, element, attrs)->
              	# ... #

          return directive

      angular
          .module('app.widgets')
          .directive('myCalendarRange', myCalendarRange)
    )()
    ```

**[Back to top](#table-of-contents)**

## Resolving Promises for a Controller

  - **Controller Activation Promises**: Resolve start-up logic for a controller in an `activate` function.

    *Why?*: Placing start-up logic in a consistent place in the controller makes it easier to locate, more consistent to test, and helps avoid spreading out the activation logic across the controller.

    ```coffeescript
    ### avoid ###
    (->
      Avengers = (dataservice)->
          init = ()=>
            @avengers = []
            @title = 'Avengers'

          dataservice
            .getAvengers()
            .then (data)=>
              @avengers = data

          init()
          return

      angular
        .module('app')
        .controller('Avengers', Avengers)
    )()
    ```

    ```coffeescript
    ### recommended ###
    (->
      Avengers = (dataservice)->

        init = ()=>
          @avengers = []
          @title = 'Avengers'

          activate()

        ##############

        activate = ()=>
          dataservice
            .getAvengers()
            .then (data)=>
              @avengers = data

        init()
        return

      angular
        .module('app')
        .controller('Avengers', Avengers)
    )()
    ```

  - **Route Resolve Promises**: When a controller depends on a promise to be resolved, resolve those dependencies in the `$routeProvider` before the controller logic is executed.

    *Why?*: A controller may require data before it loads. That data may come from a promise via a custom factory or [$http](https://docs.angularjs.org/api/ng/service/$http). Using a [route resolve](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider) allows the promise to resolve before the controller logic executes, so it might take action based on that data from the promise.

    ```coffeescript
    ### avoid ###
    (->

      Avengers = (movieService)->

        // unresolved
        @movies
        // resolved asynchronously
        movieService
          .getMovies()
          .then (response)=>
            @movies = response.movies;

        return

      angular
        .module('app')
        .controller('Avengers', Avengers)
    )()
    ```

    ```coffeescript
    ### better ###

    # route-config.js
    (->

      config = ($routeProvider)->
        $routeProvider
          .when '/avengers',

            templateUrl: 'avengers.html'
            controller: 'Avengers as vm'

            resolve:
              moviesPrepService: (movieService)->
                movieService.getMovies()

      angular
        .module('app')
        .config(config)
    )()

    # avengers.js
    (->
      Avengers = (moviesPrepService)->

        @movies = moviesPrepService.movies

        return

      angular
        .module('app')
        .controller('Avengers', Avengers);
    )()

    ```

**[Back to top](#table-of-contents)**

## Manual Dependency Injection

  - **UnSafe from Minification**: Avoid using the shortcut syntax of declaring dependencies without using a minification-safe approach.

      *Why?*: The parameters to the component (e.g. controller, factory, etc) will be converted to mangled variables. For example, `common` and `dataservice` may become `a` or `b` and not be found by AngularJS.

    ```coffeescript
    ### avoid - not minification-safe ###
    (->
      Dashboard = (common, dataservice)->

      angular
        .module('app')
        .controller('Dashboard', Dashboard)
    )()
    ```

    - This code may produce mangled variables when minified and thus cause runtime errors.

    ```javascript
    ### avoid - not minification-safe ###
    angular.module('app').controller('Dashboard', d);function d(a, b) { }
    ```


  - **Manually Identify Dependencies**: Use $inject to manually identify your dependencies for AngularJS components.

      *Why?*: This technique mirrors the technique used by [`ng-annotate`](https://github.com/olov/ng-annotate), which I recommend for automating the creation of minification safe dependencies. If `ng-annotate` detects injection has already been made, it will not duplicate it.

      *Why?*: This safeguards your dependencies from being vulernable to minification issues when parameters may be mangled. For example, `common` and `dataservice` may become `a` or `b` and not be found by AngularJS.

      *Why?*: Avoid creating inline dependencies as long lists can be difficult to read in the array. Also it can be confusing that the array is a series of strings while the last item is the component's function.

    ```coffeescript
    ### avoid ###
    (->
      Dashboard = ($location, $routeParams, common, dataservice)->

      angular
        .module('app')
        .controller('Dashboard',
          ['$location', '$routeParams', 'common', 'dataservice', Dashboard])
    )()
    ```

    ```coffeescript
    ### recommended ###
    (->
      Dashboard = ($location, $routeParams, common, dataservice)->

      Dashboard
        .$inject = ['$location', '$routeParams', 'common', 'dataservice']

      angular
        .module('app')
        .controller('Dashboard', Dashboard)
    )()
    ```

**[Back to top](#table-of-contents)**

## Minification and Annotation

  - **ng-annotate**: Use [ng-annotate](//github.com/olov/ng-annotate) for [Gulp](http://gulpjs.com) or [Grunt](http://gruntjs.com) and comment functions that need automated dependency injection using `/** @ngInject */`

      *Why?*: This safeguards your code from any dependencies that may not be using minification-safe practices.

      *Why?*: [`ng-min`](https://github.com/btford/ngmin) is deprecated

  - The following code is not using minification safe dependencies.

    ```coffeescript
    (->
      ### @ngInject ###
      Avengers = (storageService, avengerService)->

        init = ()=>
          @heroSearch = ''
          @storeHero = storeHero;

        storeHero = ()=>
          hero = avengerService.find(@heroSearch)
          storageService.save(hero.name, hero)

        init()
        return

      angular
        .module('app')
        .controller('Avengers', Avengers)
    )()
    ```

  - When the above code is run through ng-annotate it will produces the following output with the `$inject` annotation and become minification-safe.

    ```coffeescript

    (->
      ### @ngInject ###
      Avengers = (storageService, avengerService)->

        init = ()=>
          @heroSearch = ''
          @storeHero = storeHero

        storeHero = ()=>
          hero = avengerService.find(@heroSearch)
          storageService.save(hero.name, hero)

        init()
        return

      Avengers
        .$inject = ['storageService', 'avengerService']
      angular
        .module('app')
        .controller('Avengers', Avengers)
    )()

    ```

  - Note: If `ng-annotate` detects injection has already been made (e.g. `@ngInject` was detected), it will not duplicate the `$inject` code.

  - Note: Starting from AngularJS 1.3 use the [`ngApp`](https://docs.angularjs.org/api/ng/directive/ngApp) directive's `ngStrictDi` parameter. When present the injector will be created in "strict-di" mode causing the application to fail to invoke functions which do not use explicit function annotation (these may not be minification safe). Debugging info will be logged to the console to help track down the offending code.
  `<body ng-app="APP" ng-strict-di>`


  - **Use Gulp or Grunt for ng-annotate**: Use [gulp-ng-annotate](https://www.npmjs.org/package/gulp-ng-annotate) or [grunt-ng-annotate](https://www.npmjs.org/package/grunt-ng-annotate) in an automated build task. Inject `/* @ngInject */` prior to any function that has dependencies.

      *Why?*: ng-annotate will catch most dependencies, but it sometimes requires hints using the `/* @ngInject */` syntax.

    - The following code is an example of a gulp task using ngAnnotate

    ```coffeescript

    gulp.task 'js', ['jshint'], ()->
        source = pkg.paths.js
        return gulp.src(source)
            .pipe(sourcemaps.init())
            .pipe(concat('all.min.js', {newLine: ';'}))
            # Annotate before uglify so the code get's min'd properly.
            .pipe(ngAnnotate({
                # true helps add where @ngInject is not used. It infers.
                # Doesn't work with resolve, so we must be explicit there
                add: true
            }))
            .pipe(bytediff.start())
            .pipe(uglify({mangle: true}))
            .pipe(bytediff.stop())
            .pipe(sourcemaps.write('./'))
            .pipe(gulp.dest(pkg.paths.dev))


    ```

**[Back to top](#table-of-contents)**

## Exception Handling

  - **decorators**: Use a [decorator](https://docs.angularjs.org/api/auto/service/$provide#decorator), at config time using the [`$provide`](https://docs.angularjs.org/api/auto/service/$provide) service, on the [`$exceptionHandler`](https://docs.angularjs.org/api/ng/service/$exceptionHandler) service to perform custom actions when exceptions occur.

      *Why?*: Provides a consistent manner in which to customize how exceptions are handled for development-time or run-time.

	```coffeescript

  (->

    exceptionConfig = ($provide)->
        $provide.decorator
            '$exceptionHandler', ['$delegate', '$log', extendExceptionHandler]


    extendExceptionHandler = ($delegate, $log)->
        return (exception, cause)->
            $delegate(exception, cause)
            errorData =
              exception: exception,
              cause: cause

            msg = 'ERROR PREFIX' + exception.message
            $log.error(msg, errorData)

            # Log during dev with http://toastrjs.com
            # or any other technique you prefer
            toastr.error(msg)


    angular
        .module('app.exception')
        .config(['$provide', exceptionConfig])
  )()
	```

**[Back to top](#table-of-contents)**

## Naming
TODO

**[Back to top](#table-of-contents)**

## Application Structure
TODO

**[Back to top](#table-of-contents)**

## Modularity
TODO

**[Back to top](#table-of-contents)**

## Angular $ Wrapper Services

  - **$document and $window**: Use [`$document`](https://docs.angularjs.org/api/ng/service/$document) and [`$window`](https://docs.angularjs.org/api/ng/service/$window) instead of `document` and `window`.

    *Why?*: These services are wrapped by Angular and more easily testable than using document and window in tests. This helps you avoid having to mock document and window yourself.

  - **$timeout and $interval**: Use [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) and [`$interval`](https://docs.angularjs.org/api/ng/service/$interval) instead of `setTimeout` and `setInterval` .

    *Why?*: These services are wrapped by Angular and more easily testable and handle AngularJS's digest cycle thus keeping data binding in sync.

**[Back to top](#table-of-contents)**

## Comments

  - **jsDoc**: If planning to produce documentation, use [`jsDoc`](http://usejsdoc.org/) syntax to document function names, description, params and returns

    *Why?*: You can generate (and regenerate) documentation from your code, instead of writing it from scratch.

    *Why?*: Provides consistency using a common industry tool.

    ```coffeescript
    (->
     ###
     # @name logger
     # @desc Application wide logger
     ###
      logger = ($log)->

        return
         ###
         # @name logError
         # @desc Logs errors
         # @param {String} msg Message to log
         # @returns {String}
         ###
          logError: (msg)->
            loggedMsg = 'Error: ' + msg
            $log.error(loggedMsg)
            return loggedMsg

      angular
        .module('app')
        .factory('logger', logger);
    )()
    ```

**[Back to top](#table-of-contents)**

## JS Hint

  - **Use an Options File**: Use JS Hint for linting your JavaScript and be sure to customize the JS Hint options file and include in source control. See the [JS Hint docs](http://www.jshint.com/docs/) for details on the options.

    *Why?*: Provides a first alert prior to committing any code to source control.

    *Why?*: Provides consistency across your team.

    ```javascript
    {
        "asi": true,
        "boss": false,
        "browser": true,
        "camelcase": true,
        "curly": true,
        "eqeqeq": true,
        "eqnull": false,
        "es5": false,
        "expr": false,
        "evil": false,
        "forin": false,
        "globals": {
            "angular": true,
            "toastr": true,
            "breeze": true,
            "moment": true,
            "q": true
        },
        "immed": true,
        "indent": 4,
        "latedef": "nofunc",
        "loopfunc": false,
        "maxdepth": 4,
        "maxlen": 120,
        "maxparams": 10,
        "newcap": true,
        "noarg": true,
        "node": true,
        "noempty": true,
        "nomen": false,
        "nonew": true,
        "onevar": false,
        "passfail": false,
        "plusplus": false,
        "quotmark": "single",
        "shadow": false,
        "strict": false,
        "sub": true,
        "supernew": false,
        "trailing": true,
        "undef": false,
        "unused": false,
        "validthis": true
    }
    ```

**[Back to top](#table-of-contents)**

## AngularJS docs
For anything else, API reference, check the [Angular documentation](//docs.angularjs.org/api).

## Contributing

Open an issue first to discuss potential changes/additions. If you have questions with the guide, feel free to leave them as issues in the repo. If you find a typo, create a pull request. The idea is to keep the content up to date and use github’s native feature to help tell the story with issues and PR’s, which are all searchable via google. Why? Because odds are if you have a questions, someone else does too! You can learn more here at about how to contribute.

## License

#### (The MIT License)

Copyright (c) 2014 [John Papa](http://johnpapa.net)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**[Back to top](#table-of-contents)**
