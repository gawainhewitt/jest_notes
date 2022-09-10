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

### Testing environment

By default you will be testing in the Node.js environment. For a web app you can specify that you want to test in a browser like environment using the the following docblock.

To to use @jest environment you will need to install it first using `npm i jest-environment-jsdom`

```
/**
 * @jest-environment jsdom
 */
 ```

 ### Accessing files in your test

 Use the fs module to access files. 

 We use this to load html for example for behavioural tests

 You need to require it before you can use it.

 ```
 const fs = require('fs');
 ```
