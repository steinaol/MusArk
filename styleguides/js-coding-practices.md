# Pre Merge Request

## Run
- npm run lint
- npm run flow
- npm run formatcode
- npm test a

# While coding

## Naming
- We split currently files into smart components (Container) and pure view components (Component)
 - ex:
 - SearchObjectContainer.js
 - SearchObjectComponent.js
 - Both Container and Component files should have first character uppercased.

# Side effects
- We dont call hashHistory (or other browser history) from pure view components.
 - instead send in a goToUrl property from the smart component 
    ```
        const props = {
          // ...
          goToUrl: hashHistory.push,
          // ....
        };
    ```

# Flow type checking
- We enable flow type checking in our tests as well as in our code.
- No changes on old code should be left without flow unless its stupendeous difficult to add.
