Details for Codacy coverage

Login to https://app.codacy.com with musit-project user(use KeePass for user details)

Get the secure CODACY_PROJECT_TOKEN for the project (https://github.com/codacy/node-codacy-coverage)

Add the Token to the gitlab project Secret variables. Example:  https://gitlab.com/MUSIT-Norway/musit-frontend/settings/ci_cd

Make sure below code in frontend package.json 

```
"codacy-coverage": "^2.0.2",
...
...
"codacy": "cat coverage/lcov.info | ./node_modules/.bin/codacy-coverage",
"coverage": "npm test a -- --coverage --collectCoverageFrom='src/!(_poc*)**/*.{js,jsx}'",
```

