# The repository contains an link protocol in Russian and English, as well as a CLI for managing links.

# How to Run the CLI

1. Navigate to the directory containing the `assoc` file.

2. Make it executable:

```bash
chmod +x assoc
```

3. Add it to your PATH (to run it simply as `assoc`):

`sudo mv assoc /usr/local/bin/`

4. Usage examples:

```bash
assoc init                  # initialize
assoc create-self           # create a standalone object
assoc list
assoc connect 1 1           # create a link between existing objects
assoc modify 5 --from-id 2 --to-id 3
assoc delete 4
assoc status
```

The database is saved as `associations.json` in the current directory.