---
layout: post
title: Running Rust on the Rumprun unikernel
published: false
---

The [Rust](https://www.rust-lang.org/) programming language has been able to
run on bare-metal
[without the standard library](https://github.com/charliesome/rustboot)
for quite some while now. However, most Rust applications depend on the
[`std`](https://doc.rust-lang.org/std/) crate, and therefore still need a full
operating system to run.

This is where the
[Rumprun unikernel platform](http://repo.rumpkernel.org/rumprun) comes into
play. It allows you to build your POSIX applications into bootable
single-purpose images. Because unikernels are tailored to run a single application,
they come without the footprint of a full-featured operating system. This makes
them a great tool for application virtualization. Supported platforms of Rumprun include not
only Xen/EC2 and KVM, but you can also run your image on bare-metal hardware.

Rumprun is based on [rump kernels](http://rumpkernel.org), it reuses
NetBSD's libc and drivers as components to provide a POSIX-y interface --
the interface which the Rust standard library is built upon.

For the last couple of days we have been working on Rumprun support for Rust --
you can now deploy your Rust application as a Rumprun unikernel. With our toolchain
set up, a single [cargo](https://github.com/rust-lang/cargo/) command is all
you need to turn your Rust application into a Rumprun unikernel image.

Here is a screenshot of an example TCP/IP server written in Rust and built with
cargo, running in a Rumprun unikernel on QEMU.

[<img alt="An example Rust TCP/IP server on Rumprun" src="/public/rust-on-rumprun/tcp.png">](/public/rust-on-rumprun/tcp.png)

Note that running Rust on bare-metal without a standard library is different
from running Rust on bare-metal Rumprun: Rust's
[`#![no_std]`](https://doc.rust-lang.org/book/no-stdlib.html) feature
means you write your own drivers and environment. Rust on Rumprun on the other hand already
provides you with production-quality NetBSD drivers and a standard library. You can
choose which subsystems and drivers you want to compile into your image.

A significant portion of the work was spent on making `std` work correctly
on NetBSD. As a side effect of this project, you can now also compile
your Rust binaries for [NetBSD/amd64](https://github.com/rust-lang/rust/pull/28543).

### Getting started

The remaining part of this blog post briefly guides you through the necessary
steps to build your Rust application into a Rumprun unikernel image. This is
what you have to do:

 - Build the Rumprun unikernel platform, it provides the tools and libraries
   to build Rumprun unikernels.
 - Set up a Rust cross-compiler and standard library for Rumprun.
   This allows you to compile binaries that you can later "bake" into a Rumprun
   unikernel.
 - Compile your Rust application using the Rust cross-compiler.
 - Bake the generated Rust binary into a Rumprun unikernel image, which you
   can then run on a hypervisor or bare-metal machine of your choice.

#### Building Rumprun

First, we need to build Rumprun. The repository provides its own cross-compile
tools that are needed in the following steps.
Check out the excellent [Tutorial - Building Rumprun Unikernels](http://wiki.rumpkernel.org/Tutorial%3A-Building-Rumprun-Unikernels)
for instructions. I recommend following the whole tutorial, but if you are in a
hurry, the section "Building the Rumprun Platform" is all you need to continue
to the next step.

#### Building a Rust cross-compiler

The Rust source tree already
[contains all patches](https://github.com/rust-lang/rust/pull/28593) needed
to build for Rumprun, but you will have to enable the Rumprun target.
We provide the scripts for building a Rust cross-compiler and the `std`
crate automatically in the  [rumprun-packages](http://repo.rumpkernel.org/rumprun-packages)
repository. Just follow the [instructions on how to build the Rust package](https://github.com/rumpkernel/rumprun-packages/blob/master/rust/README.md).

#### Compiling and baking your Rust application

If everything is working properly, building your Rust application as a
unikernel image is as easy as running `cargo rumpbake`. Be aware though that
at the time of writing, some crates on [crates.io](https://crates.io) don't
have proper NetBSD support yet, and might fail to compile or execute.

*Congratulations!* You are now set up to build your own Rust applications into
Rumprun unikernel images.

### Acknowledgments

This project is all about taking existing pieces of software and putting
them together. Without the awesome previous work of the Rust and the rump kernel
hackers, this could not exist. A special thank you goes to
[@anttikantee](https://twitter.com/anttikantee) for his support along the way.
