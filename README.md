## Dumb example

``` es6
// lib.js
function filewrite(content, filepath) {
  open(filepath).write(content);
}

function getFromDB(tableName, key) {
  // do magic db things
  return someData;
}

function getBlogpost(fromDb, blogpostName) {
  let blogpostBody = fromDb('blogpost', blogpostName);
  return wrapBlogpost(blogpostBody);
}

function wrapBlogpost(body) {
  return "<div class='article'>" + body + "</div>"return
}

```

``` es6
import * from 'lib';

// runapp.js
// untested (we trust that ES6 classes work as advertised and
// we don't have our own logic in our constructor
let filewriter = FileWriter();

// untested (this is just composition of the things that are under test)
filewrite(getBlogpost(getFromDB, 'mypost'), 'post.html');
```

## Push this stuff to the boundary of your abstractions

* Examples of stuff that doesn't change much:
  * how writing to a file works
  * how talking to a DB works
  * how printing output to STDOUT works
  * how to make an HTTP request
  * how to make a secure socket connection

* This stuff should be:
  * Tested involving all the parties involved (a file, the file system, the code
    handles opening closing the file, etc)
  * Tests for this stuff may involve temporary resources
  * Temporary resources created by these tests may need to be cleaned up after

* What you're testing:
  * The side effect happened (hey there's a file in the file system!)

* What you're not testing:
  * Data encapsilated in the side effect (hey there's data that looks like this
    in the file)

``` es6
let expected = "any junk. I don't care what's here"

filewriter.write(expected, 'somepath.txt')
assert(() => {
  return filereader.read('somepath.txt') == expected
})

// We don't care how we got the text, what the text looks like, etc.
// We only care that a file exists at the correct path and it has
// the content we wanted to put in it.

// note: we *needed* the filesystem for the test and it needed to be
// a filesystem that's at least symantically *compatible* with the filesystem
// we'll use in production.
```

``` es6
// a database needs to be present, it needs some data in it and it needs to be
// where we want it in the database for the test. We don't care about what
// the data looks like, what we're going to do with it, where it came from...
// just that we're able to find it where we expect and it's what we put into the
// the DB.
magicallySpinUpDB(someFixtureData);

// the same data from the DB is loaded from some static place.
// this is the *same* place we got the data from to put it in the
// database.
let expected = someFixtureData;

assert(() => {
  return getFromDB('ourTable', 'ourKey') == expected;
})
```

## Make this the core of your abstractions

* Stuff that changes constantly:
  * what you write to a file
  * who can write to a file
  * what you write to a DB
  * who can write to a DB
  * what you print to STDOUT
  * what a particular HTTP request looks like

* This stuff should be:
  * referentially transparent (data in / data out (functions can be data!))
  * tested with data structures that match real life
  * tested using only code and data (no side effects)

* What you're testing:
  * When I put X data in I get Y data out.

* What you're not testing:
  * The side effect happened (hey there's a file in the file system!)

``` es6
assert(() => {
  let body = 'we don't care what';
  let expected = "<div class='article'>" + body + "</div>";
  return wrapBlogpost(body) == expected;
})
```

``` es6
assert(() => {
  let fixtureText = "we don't care what";
  let expected = wrapBlogpost(fixtureText);
  let fakeGetFromDB = (table, key) => {
    if (table == 'blogposts' && key == 'myBlogpost') {
      return fixtureText
    }
  });

  return getBlogpost(fakeGetFromDB, 'myBlogPost') == expected;
  // asserts getBlogpost is a composition of wrapBlogpost and
  // a passed in function to get the content with an interface
  // that accepts a table and key and returns data based
  // on that table and key
})
```
