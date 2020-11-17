# Pipe Gifts

This is a pair of scripts designed to transfer files between users on a single node.

It's intended to have a UI a *bit* like magic-wormhole, in that the person giving the file gets a generated credential they have to pass out-of-band to to the person receiving the file.


Behind the scenes, it creates a pair of named pipes in a public temporary directory, which it uses to authenticate and then pipe a streaming `tar`-pair through.

The users have to be logged into the same node at the same time, and since the pipes depend on streaming it's all synchronous in terms of presence, but there's no stored state.

## To Do:

 - better/any docs
 - optional encryption for data-stream
 - better credential-handling mechanism (use `openssl enc` to encrypt known data with the password?)
 - think about safety -- make tar refuse to overwrite files?
 - tweak transfer output to be a little more explicative

## Potential Problems:

 - not really secure


## Example

Sender:

```
# Make a junk 1GiB file
$ dd if=/dev/zero of=some_file bs=1024MiB count=1
$ pipe-give some_file
Your transfer ID: qBMqUIOt
   and password: 704122286

Please enter this at receiver.

Remember you must be on the same node.
[Ctrl-C to cancel.]
```

Receiver:
```
$ $ ./pipe-receive
Please enter your transfer ID: qBMqUIOt
Please enter transfer password: 704122286
```

Sender (`tar v` output):
```
some_file
```

Receiver (`tar v` output):
```
some_file
```

