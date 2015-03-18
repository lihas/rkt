# Rocket - App Container runtime

[![Build Status](https://travis-ci.org/coreos/rocket.png?branch=master)](https://travis-ci.org/coreos/rocket)
[![godoc](https://godoc.org/github.com/coreos/rocket?status.svg)](http://godoc.org/github.com/coreos/rocket)

_Release early, release often: Rocket is currently a prototype and we are seeking your feedback via issues and pull requests_

Rocket is a CLI for running app containers, and an implementation of the [App Container Spec](Documentation/app-container.md). The goal of Rocket is to be composable, secure, and fast.

[Read more about Rocket in the launch announcement](https://coreos.com/blog/rocket).

![Rocket Logo](logos/rocket-horizontal-color.png)

## Trying out Rocket

The CLI for Rocket is called `rkt`, and is currently supported on amd64 Linux. A modern kernel is required but there should be no other system dependencies. We recommend booting up a fresh virtual machine to test out Rocket.

To install the `rkt` binary, grab the latest release directly from GitHub:

```
wget https://github.com/coreos/rocket/releases/download/v0.4.1/rocket-v0.4.1.tar.gz
tar xzvf rocket-v0.4.1.tar.gz
cd rocket-v0.4.1
./rkt help
```

Keep in mind while running through the examples that right now `rkt` needs to be run as root for most operations.

## Rocket basics

### Downloading an App Container Image (ACI)

Rocket uses content addressable storage (CAS) for storing an ACI on disk. In this example, the image is downloaded and added to the CAS.

Since Rocket verifies signatures by default, you will need to first [trust](https://github.com/coreos/rocket/blob/master/Documentation/signing-and-verification-guide.md#establishing-trust) the [CoreOS public key](https://coreos.com/dist/pubkeys/aci-pubkeys.gpg) used to sign the image:

```
$ sudo rkt trust --prefix coreos.com/etcd
Prefix: "coreos.com/etcd"
Key: "https://coreos.com/dist/pubkeys/aci-pubkeys.gpg"
GPG key fingerprint is: 8B86 DE38 890D DB72 9186  7B02 5210 BD88 8818 2190
  CoreOS ACI Builder <release@coreos.com>
Are you sure you want to trust this key (yes/no)? yes
Trusting "https://coreos.com/dist/pubkeys/aci-pubkeys.gpg" for prefix "coreos.com/etcd".
Added key for prefix "coreos.com/etcd" at "/etc/rkt/trustedkeys/prefix.d/coreos.com/etcd/8b86de38890ddb7291867b025210bd8888182190"
```

A detailed, step-by-step guide for the signing procedure [is here](Documentation/getting-started-ubuntu-trusty.md#trust-the-coreos-signing-key).

Now that we've trusted the CoreOS public key, we can fetch the ACI:

```
$ sudo rkt fetch coreos.com/etcd:v2.0.4
rkt: searching for app image coreos.com/etcd:v2.0.4
rkt: fetching image from https://github.com/coreos/etcd/releases/download/v2.0.4/etcd-v2.0.4-linux-amd64.aci
Downloading aci: [==========================================   ] 3.47 MB/3.7 MB
Downloading signature from https://github.com/coreos/etcd/releases/download/v2.0.0/etcd-v2.0.4-linux-amd64.aci.asc
rkt: signature verified: 
  CoreOS ACI Builder <release@coreos.com>
sha512-1eba37d9b344b33d272181e176da111e
```

These files are now written to disk:

```
$ find /var/lib/rkt/cas/blob/
/var/lib/rkt/cas/blob/
/var/lib/rkt/cas/blob/sha512
/var/lib/rkt/cas/blob/sha512/1e
/var/lib/rkt/cas/blob/sha512/1e/sha512-1eba37d9b344b33d272181e176da111ef2fdd4958b88ba4071e56db9ac07cf62
```

Per the [App Container Specification](https://github.com/appc/spec/blob/master/SPEC.md#image-archives), the SHA-512 hash is of the tarball and can be reproduced with other tools:

```
$ wget https://github.com/coreos/etcd/releases/download/v2.0.0/etcd-v2.0.0-linux-amd64.aci
...
$ gzip -dc etcd-v2.0.0-linux-amd64.aci > etcd-v2.0.0-linux-amd64.tar
$ sha512sum etcd-v2.0.0-linux-amd64.tar
1eba37d9b344b33d272181e176da111ef2fdd4958b88ba4071e56db9ac07cf62cce3daaee03ebd92dfbb596fe7879938374c671ae768cd927bab7b16c5e432e8  etcd-v2.0.4-linux-amd64.tar
```

### Launching an ACI

After it has been retrieved and stored locally, an ACI can be run by pointing `rkt` at either the ACI's hash or URL.

```
# Example of running via ACI hash
$ sudo rkt run sha512-1eba37d9b344b33d272181e176da111e
...
Press ^] three times to kill container
```

```
# Example of running via ACI URL
$ sudo rkt run https://github.com/coreos/etcd/releases/download/v2.0.4/etcd-v2.0.4-linux-amd64.aci
...
Press ^] three times to kill container
```

In the latter case, `rkt` will do the appropriate ETag checking on the URL to make sure it has the most up to date version of the image.

Note that the escape character ```^]``` is generated by ```Ctrl-]``` on a US keyboard. The required key combination will differ on other keyboard layouts. For example, the Swedish keyboard layout uses ```Ctrl-å``` on OS X and ```Ctrl-^``` on Windows to generate the ```^]``` escape character.

## Contributing to Rocket

Rocket is an open source project under the Apache 2.0 [license](LICENSE), and contributions are gladly welcomed!
See the [Hacking Guide](Documentation/hacking.md) for more information on how to build and work on Rocket.
See [CONTRIBUTING](CONTRIBUTING.md) for details on submitting patches and the contribution workflow.

## Contact

- Mailing list: [rocket-dev](https://groups.google.com/forum/?hl=en#!forum/rocket-dev)
- IRC: #[coreos](irc://irc.freenode.org:6667/#coreos) on freenode.org
- Planning/Roadmap: [milestones](https://github.com/coreos/rocket/milestones)
