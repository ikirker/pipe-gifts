# Pipe Gifts

This is a pair of scripts designed to transfer files between users on a single node.

It's intended to have a UI a *bit* like magic-wormhole, in that the person giving the file gets a generated credential they have to pass out-of-band to to the person receiving the file.


Behind the scenes, it creates a pair of named pipes in a public temporary directory, which it uses to authenticate and then pipe a streaming `tar`-pair through.

The users have to be logged into the same node at the same time, and since the pipes depend on streaming it's all synchronous in terms of presence, but there's no stored state.

## Maybe To Do:

 - optional encryption for data-stream (not sure how this would affect transfer rate)
 - better credential-handling mechanism (use `openssl enc` to encrypt known data with the password?)
 - checksums?
 - option to specify ID and password on the receiving command-line?
 - some kind of diceware-style algorithm to make the IDs and passwords easier to tell someone?

## Potential Problems:

 - not really secure, but maybe doesn't matter
 - no file integrity checks

## Speed

In quick testing, I managed about 250MiB/s from Lustre to Lustre, and about 1GiB/s from tmpfs to tmpfs. It's not ideal, but it's fast enough for a lot of uses.

## Example

##### Sender

The Sender makes a test file to send to their friend:

```
# Make a junk 1GiB file
$ dd if=/dev/zero of=some_file bs=1GiB count=1
```

And then uses `pipe-give` to send it:

```
$ pipe-give some_file

Your transfer ID: dzbGPpE3
    and password: 688815576

Please enter this at receiver.

Remember you must be on the same node.

This node is: login13

    [Ctrl-C to cancel.]
```

The Sender gives the ID and password to Receiver...

##### Receiver

... who then runs the `pipe-receive` and enters them in, and presses `y` to confirm that the list of files is okay:

```
$ pipe-receive
Please enter your transfer ID: dzbGPpE3
Please enter transfer password: 688815576
[2020-12-02 15:06:44] pipe-receive: (info) password correct, receiving identity and file count
[2020-12-02 15:06:44] pipe-receive: (info) receiving list of files for approval

-- File List --
some_file
---------------

User coursb4 wants to copy this file to you.
Would you like to accept? [press y for yes, anything else to cancel]
[2020-12-02 15:06:55] pipe-receive: (info) confirming with sender
[2020-12-02 15:06:55] pipe-receive: (info) receiving files...
some_file
1.00GiB 0:00:04 [ 234MiB/s] [                        <=>                                                                                                                                                                                                                                ]
[2020-12-02 15:06:59] pipe-receive: (info) transfer complete
```

##### Sender

The Sender sees their file has been transferred:

```
[2020-12-02 15:06:44] pipe-give: (info) received correct password
[2020-12-02 15:06:44] pipe-give: (info) sending identity and number of files
[2020-12-02 15:06:44] pipe-give: (info) sending file listing to receiver for approval
[2020-12-02 15:06:55] pipe-give: (info) confirmation received
[2020-12-02 15:06:55] pipe-give: (info) transferring files...
some_file
1.00GiB 0:00:04 [ 234MiB/s] [                        <=>                                                                                                                                                                                                                                ]
[2020-12-02 15:06:59] pipe-give: (info) transfer complete
```

