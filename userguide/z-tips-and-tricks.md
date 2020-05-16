# Tips And Tricks

## Forging the Ording Documents

When the `README.md` file is generated, it traverses the categories and
sub-category folders in this root and create an alphabetically sorted list in
the file. It does the same for the documents, and this could be somewhat
frustrating if you have a document you want to make sure is listed first or last
for some reason. Well, good news! We can trick the ordering by specifying a
title we *know* will be last, for example.

Let's say I'm creating a 'Tips and Tricks' document for a userguide (i.e. this
file) which I specifically want to put last when it's listed in the `README.md`
file. Here's how to do it:

1. Create a new document, and make the title start with 'z'
```bash
greg:md $ ./md -n -t "z tips and tricks" -c userguide
Generating new Markdown file with title "Z Tips And Tricks" --> userguide/z-tips-and-tricks.md
```

2. As you can see the filename is `z-tips-and-trcks.md`. When we edit this file
   change the default title and simply remove the 'Z':
```markdown
# Z Tips and Tricks
```
Becomes:
```markdown
# Tips and Tricks
```

3. When the index is re-generated using the `md --index` command the ordering is
   determined by filename sorting alphabecially, not the titles.
   
In general we want the titles to match the filename but there is nothing
stopping us from purposly mis-matching them in order to force the order.
likewise we could always something like this:
```bash
greg:md $ cd userguide/
greg:userguide $ ls
installation.md                       reindexing-the-markdown-documents.md  specifying-the-document-root.md       usage.md                              z-tips-and-tricks.md
greg:userguide $ mv installation.md a-installation.md
greg:userguide $ ls
a-installation.md                     reindexing-the-markdown-documents.md  specifying-the-document-root.md       usage.md                              z-tips-and-tricks.md
greg:userguide $ 
```

Now the installation file, starting with an 'a-` will force it's ordering in the index, while the title inside it can still be "Installation"!

## Renaming Categories

Renaming categories is as simple as renaming the directory name. For example, let's say there were a catory named tips which we wanted to be tricks. Simply rename the directory and then re-generate the index:
```bash
greg:md $ ls
INSTALL.md        LICENSE           README-footer.md  README-header.md  README.md         TEMPLATE.md       introduction/     md*               tips/             userguide/
greg:md $ mv tips tricks
greg:md $ ./md --index
Generating markdown index to file: README.md
```
