# Specifying the document root

By default, `md` works on the current working directory, however if you want to
specify a different document root you can use the `--root` flag

For example:
```
md --root /local/faq/ --new "Creating a new document root" --catagory til
```

## Environment variable

Another way to set the document root is to use an environment variable `MARKDOWN_ROOT`:

```bash
export MARKDOWN_ROOT="/local/documents"
```

This would override the default, or anything passed on the CLI as `--root`
