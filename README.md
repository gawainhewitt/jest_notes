# Jest Notes

[Jest](https://jestjs.io/docs/getting-started) is testing library for JavaScript. These are my personal notes to help me as I use it. Maybe they can help you too.


### Testing environment

By default you will be testing in the Node.js environment. For a web app you can specify that you want to test in a browser like environment using jsdom.

If you've not set this up you may get the following error when running your test. For me this has been a particular problem when the file I'm testing is referencing html in the dom. 

```
ReferenceError: document is not defined
```

To to use jsdom environment in jest you will need to install it first using `npm i jest-environment-jsdom`

This docblock can be added at the top of a test file to tell that file to use jsdom. 

```
/**
 * @jest-environment jsdom
 */
```

This will not transfer this setting to the tested file however. To do that we will need to run jest with the environment set to jsdom with this command 

```
jest --env=jsdom
```

We can make this a little slicker by adding it as a script to our package.json file and then running our tests using the command `npm test`.

```
"scripts": {
    "test": "jest --env=jsdom",
  },

```

### Including our HTML in the test

Once we have global jsdom then we can then reference html within the class or module that we are testing. <br>

Within our test we will then be creating an instance of that class in this file, which means that we can read our html file using fs in the test and our test will use the html we imported to test. 

for example:

```
const fs = require("fs");
const html = fs.readFileSync("./index.html");
document.documentElement.innerHTML = html;
```

### Accessing files in your test

Use the fs module to access files. 

We use this to load html for example for behavioural tests

You need to require it before you can use it.

```
const fs = require('fs');
```

### Testing mouseenter (and mouseover)

If we are testing a button click we can just use `document.querySelector("#id").click();` to simulate a mouse click in JS. For mouseenter and mouseover and possibly others this is not possible. Instead we have to use `dispatchEvent` as follows:

```
document.querySelector("#id").dispatchEvent(
          new MouseEvent("mouseenter", {
            view: window,
            bubbles: true,
            cancelable: true
          })
        );
```

### testing selectstart and other generic events

Similar to above but:

```
document.dispatchEvent(new Event('selectstart'));
```

### config

set up a jest.config.js file and create an object within which you place configuration

```
const config = {
  verbose: true,
  collectCoverage: true,
  setupFilesAfterEnv: ["<rootDir>/tests/setupTests.js"]
};

module.exports = config;

```

### setupFilesAfterEnv

Key value pair in our config object within our jest.config.js file.

`setupFilesAfterEnv: ["<rootDir>/tests/setupTests.js"]`

The array contains files to execute before each test. In the above example setupTests.js contains the following placeholder:

```
//<rootDir>/tests/setupTests.js

beforeEach(() => {
     console.log("before each");
});

afterEach(() => {
     console.log("after each");
});

```

### Testing Api Calls

This matches the code in JS_notes.md on the same subject

```
const NotesApi = require('./notesApi');

global.fetch = jest.fn(() =>                              // a manual mock jest.fn() allows us to create a mock
  Promise.resolve({                                       // resolve first promise
    json: () => Promise.resolve(['This note is coming from the server']), //resolve second promise
  })
);

beforeEach(() => {                                        // clear mocks between tests so pristine each time
  fetch.mockClear();
});


describe('notesApi class', () => {
  it('calls fetch and loads note from server', async () => {
    const api = new NotesApi();
    let result;
    await api.loadNotes((notes) => {
      result = notes;                                      // this callback assigns the notes to the variable result above
    });
    expect(result[0]).toBe('This note is coming from the server');      //which we can then test for
    expect(fetch).toHaveBeenCalledWith('http://localhost:3000/notes');
    expect(fetch).toHaveBeenCalledTimes(1);
    })
  it('handles exception with null', async () => {
    fetch.mockImplementationOnce(() => Promise.reject('API failure'))     // overide the mock this once with a reject
    const api = new NotesApi();                                 
    let result;
    await api.loadNotes((notes) => {
      result = notes;
    });
    expect(result).toBe(null);
    expect(fetch).toHaveBeenCalledWith('http://localhost:3000/notes');
    expect(fetch).toHaveBeenCalledTimes(1);

  })
});


```

