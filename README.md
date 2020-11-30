# Pipe Gifts

This is a pair of scripts designed to transfer files between users on a single node.

It's intended to have a UI a *bit* like magic-wormhole, in that the person giving the file gets a generated credential they have to pass out-of-band to to the person receiving the file.


Behind the scenes, it creates a pair of named pipes in a public temporary directory, which it uses to authenticate and then pipe a streaming `tar`-pair through.

The users have to be logged into the same node at the same time, and since the pipes depend on streaming it's all synchronous in terms of presence, but there's no stored state.

## To Do:

 - optional encryption for data-stream (not sure how this would affect transfer rate)
 - better credential-handling mechanism (use `openssl enc` to encrypt known data with the password?)
 - checksums?
 - option to specify ID and password on the receiving command-line?
 - some kind of diceware-style algorithm to make the IDs and passwords easier to tell someone?

## Potential Problems:

 - not really secure, but maybe doesn't matter
 - no file integrity checks


## Example

Sender:

```
# Make a junk 1GiB file
$ dd if=/dev/zero of=some_file bs=1GiB count=1

$ pipe-give some_file

Your transfer ID: XESXdZMT
    and password: 985126401

Please enter this at receiver.

Remember you must be on the same node.

    [Ctrl-C to cancel.]
```

Sender now gives ID and password to receiver, who enters them at the prompts:

```
$ pipe-receive
Please enter your transfer ID: XESXdZMT
Please enter transfer password: 985126401
```

And then the copy starts, the sender sees:

```
[2020-11-25 00:59:53] pipe-give: (info) correct password received, transferring files...
some_file
1.00GiB 0:00:01 [ 977MiB/s] [         <=>                                              ]
[2020-11-25 00:59:54] pipe-give: (info) transfer complete
```

The receiver sees:

```
[2020-11-25 00:59:53] pipe-receive: (info) password correct, receiving files...
some_file
1.00GiB 0:00:01 [ 977MiB/s] [         <=>                                              ]
[2020-11-25 00:59:54] pipe-receive: (info) transfer complete
```

