# Re-indexing the markdown documents

In order to generate (or re-generate) the index document (by default the
`README.md` file at the top of the document root) simply run `md` with the
argument `--index` or `-i` and no argument.

```
md --index
```

This will pick up all new and existing markdown files per catagory and
sub-catagory and create the tree (similar to a Table of Contents). This index
will be alphabetically sorted and organized hierarchically
