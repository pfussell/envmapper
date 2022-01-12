# envmapper

### Introduction

Once we have access to a network, we can execute arbitrary commands on discovered machines. In large organizations with many computers, it can be difficult to decide where the most fruitful targets would be. A list of processes a computer is running can tell us some important information about how the computer is used. If we see Photoshop in a list of processes, we might infer that the user is a graphic designer or photographer. If we see cmd.exe, it might be a developer or administrator. A user running excel.exe might be a manager or analyst. Any one of these might be a source of valuable business data for white-hats and black-hats alike.

### First Steps

After running tasklist.exe on every workstation and saving the results to a local directory, it's easy enough to grep through every file and find specific processes, but what if we don't even know what we're looking for? Every organization is different and our goals might be different on each penetration test so we need a way to get some high level data about all of the computers. We wrote a command-line tool called netographer (**note: the name isn't very good, what should we call it?**) which can provide us such information and more.

### users/groups

The utility enum4linux enumerates all of the users in a domain and the groups to which they are assigned. Netographer can parse this file and return a JSON object mapping either users to an array of groups they are in or groups to an array of users in that group. These are the users and groups commands respectively. Because we don't always have time or permission to run enum4linux, we can also use a list of files showing logged-in-users as the source data for this command (but we will be missing group membership).

### processes

Once we've looked at the users and groups for anything interesting, we can get a high-level view of all running processes on the network. The processes command will parse input tasklists and return a JSON object containing an array of all processes and how many times they occurred (called "process-counts"), an array of all processes and their frequency from 0 to 1 (called "process-frequencies"), and a sub-object representing all users and their running tasks (called "users"). If the output from the users command is piped into this command, the set of users will be restricted to those from the piped users object. Otherwise, the users will be the set of users from every tasklist.

### cluster

This is the fun one. The output from the previous commands is useful, but in very largs organizations even reading through those can be unmanagable. What if we could group users by the processes they are running and then invesitage each group? Two users running devenv.exe (Visual Studio), are probably doing similar kind of work and have access to similar kinds of business data. Two users running dwm.exe (Desktop Windows Manager) are probably not very similar because almost every Windows computer has that process.

With this in mind, we designed a distance metric that compares the processes in common between to users weighted by overall process frequency and places similar users closer together. Using heirarchical cluster, we initially assign every user to their own cluster. Repeatedly combining the most similar clusters reduces the number of clusters to the point where we have maybe 3 or 5 groups of hopefully different types of users based solely on the processes they were running. The inputs to this command is the output from the processes command. The output from this command is a JSON object of cluster compositions from 2 to 10 clusters showing the most common processes for each calculated cluster of users. The optimal number of clusters depends on the data so several different values are provided for convenience.


## usage
The single executable Python 3 script should be called like this for a list of users and their groups:

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
