# angular-gsapify-router

Angular UI-Router animation directive allowing configuration of state transitions using [GSAP](http://www.greensock.com/gsap-js/)

## Demo

[http://homerjam.github.io/angular-gsapify-router/](http://homerjam.github.io/angular-gsapify-router/)

## Installation

`$ npm i -S angular-gsapify-router`

## Usage

First include [TweenMax](http://greensock.com/tweenmax) (part of GSAP) on your page or in your build

In your main app file eg. `app.js`:

```javascript

// Setup dependencies
angular.module('myApp', ['ui-router', 'hj.gsapifyRouter'])

// Configure app
.config(function($stateProvider, gsapifyRouterProvider) {

	// Set default transitions to use if unspecified
	gsapifyRouterProvider.defaults = {

        // name of transition to use or 'none'
        enter: 'fadeDelayed',

        leave: 'fade'

    };

	// Add a new transition
	gsapifyRouterProvider.transition('slideToFromRight', {

        // transition duration
        duration: 0.5,

        // transition delay
        delay: 0.5,

        // start/end point for transition (eg. off screen)
        css: {
            x: '100%'
        }

	});

	// Configure states
    $stateProvider.state('home', {
        url: '/',
        views: {
            main: {
                templateUrl: '/templates/home.html',
                controller: 'HomeCtrl as home'
            },
            header: {
                templateUrl: '/templates/menu.html',
                controller: 'MenuCtrl as menu'
            }
        },
        data: {

            // define transitions per view in the data object of state using
            // `gsapifyRouter.VIEWNAME` dot notation allows inheritance/overwriting
            // of transition preferences
            'gsapifyRouter.main': {

                // when entering this state
                enter: {

                    // use this transition on the incoming view
                    in: {

                        // name of transition to use
                        transition: 'slideToFromRight',

                        // (optional) priority to determine whether to use transition
                        // of entering or leaving state (highest wins)
                        priority: 1

                    },

                    // use this transition on the outgoing view
                    out: {
                        transition: 'slideToFromLeft',
                        priority: 2
                    }

                },

                // when leaving this state
                leave: {

                     // use this transition on the incoming view
                    in: {
                        transition: 'slideToFromLeft'
                    },

                     // use this transition on the outgoing view
                    out: 'slideToFromRight'

                },
            }

            'gsapifyRouter.header': {

                enter: {

                    // use a function to determine transition
                    // note: `$state.history` is populated by `gsapifyRouter` module, not `ui-router`
                    in: ['$state', function($state) {
                        return {
                            transition: $state.current.name === $state.history[$state.history.length - 2].name ? 'reverse' : 'normal',
                            priority: 1
                        };
                    }],

                    out: {
                        transition: function(someService) {
                            return someService.getTransition('header');
                        },
                        priority: 2
                    }

                },

                leave: {

                    // define a transition directly in the state config
                    in: {
                        transition: {
                            duration: 1,
                            ease: 'Quart.easeInOut',
                            css: {
                                y: '100%'
                            }
                        },

                        // add event listener to $rootScope to trigger transition
                        trigger: 'viewInitialised'
                    },

                },

            }

        }
    });

});

```

In your templates:
```html
<!-- add 'gsapify-router' class to ui-view element -->
<div ui-view="main" class="gsapify-router"></div>
```

#### scrollRecall directive
Optionally add `scrollRecall` directive to remember and return to scroll position of previous state:
```html
<!-- add scrollRecall directive -->
<div ui-view="main" class="gsapify-router" scroll-recall></div>
```

## FAQ

#### My views jump around when the transition occurs, WTF?!

This happens because during the transition the incoming and outgoing views both exist within the dom as sibling nodes. One solution is to use absolute/fixed positioning, otherwise you can try adding the following to your css.

```css
.gsapify-router-in-setup {
    display: none;
}
```
