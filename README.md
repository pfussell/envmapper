# netographer

## usage
The single executable Python 3 script, netographer, should be called like this for a list of users and their groups:

```bash
./netographer users <enum4linux_output>
```

and like this for a list of groups and users therein:

```bash
./netographer groups <enum4linux_output>
```

and like this for a list of all tasks and their frequency:

```bash
./netographer task-frequency <tasklists_directory>
```

The result are returned as JSON objects. You can pipe the output into jq like this to query and pretty-print it:

```bash
./netographer <args> | jq '.'
```

## future
- We will want to filter processes running on any single computer and processes running on all computers (or a large percentage thereof).
- After that, we can use k-nearest neighbors and other unsupervised classification algorithms to associate users by their set of running processes.
