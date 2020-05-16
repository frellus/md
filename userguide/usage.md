# Usage

## Creating a New Document
1. change to the root of where your documentation is being written to
2. create a new document like this:
```
md --new "Create a new Document" --catagory "userguide"
```
This would do the following:
* creates a directory called 'userguide' in this documentation root, if it didn't
  exist

* creates a file called `userguide/create-a-new-document.md` with the first line being
  the title you passed on the command line, properly capitalized and formatted,
  like this:
```
# Create a New Document
```

3. The document index is then (re-)generated (by default the `README.md` file at
   the top of the document rooot)
