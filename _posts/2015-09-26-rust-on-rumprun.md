---
layout: post
title: Running Rust on the Rumprun unikernel
published: false
---

The [Rust](https://www.rust-lang.org/) programming language has been able to
run on bare-metal
[without the standard library](https://doc.rust-lang.org/book/no-stdlib.html)
for quite some while now. However, most Rust applications depend on the
[`std`](https://doc.rust-lang.org/std/) crate, and therefore still need a full
operating system to run.

This is where the 
[Rumprun unikernel platform](http://repo.rumpkernel.org/rumprun) comes into
play. It allows you to build your POSIX applications into bootable
single-purpose images. Because unikernels are tailored to run a single
application, they come without the footprint of a full-featured operating system.
Supported platforms of Rumprun include Xen/EC2 and KVM, but you can also run
your image on bare-metal hardware.

In the last couple of days we have been working on Rumprun support for
Rust -- you can now deploy your Rust application as a Rumprun unikernel. 

Rumprun is based on [rump kernels](http://rumpkernel.org), it reuses
NetBSD's libc and drivers as components to provide a POSIX-y interface.
Being a unikernel, you can choose the subsystems and drivers that you want to
compile into your image. Since there is only one application running, there is
no support for virtual memory or spawning processes.

Here is a screenshot of an example TCP/IP server written in Rust and built with
cargo, running in a Rumprun unikernel on QEMU:

[<img alt="An example Rust TCP/IP server on Rumprun" src="/public/rust-on-rumprun/tcp.png">](/public/rust-on-rumprun/tcp.png)

A significant portion of the work was spent on making `std` work correctly
on NetBSD. As a side effect of this weekend project, you can now also compile
your Rust binaries for NetBSD/amd64.

### Getting started

The remaining part of this blog post briefly guides you through the necessary
steps to build your Rust application into a Rumprun unikernel image.

#### Building Rumprun

First, we need to build Rumprun. The repository provides its own cross-compile
tools that are needed for the following steps. Note that the Rumprun target
in Rust currently only supports *x86-64*.

Check out the excellent [Tutorial - Building Rumprun Unikernels
](http://wiki.rumpkernel.org/Tutorial%3A-Building-Rumprun-Unikernels)
for instructions. I recommend following the whole tutorial, but if you are in a
hurry, the section "Building the Rumprun Platform" is all you need to continue
to the next step.

#### Building a Rust cross-compiler

The Rust source tree already contains all patches needed to build for Rumprun.
However, even if you have a recent Rust compiler installed in your system, you
still need to cross-compile the standard library for Rumprun.

The [rumprun-packages](http://repo.rumpkernel.org/rumprun-packages)
repository contains the necessary scripts to set up a Rust cross-compiler
and `std` crate for Rumprun from source, just follow 
[the provided instructions](https://github.com/rumpkernel/rumprun-packages/blob/rust/rust/README.md).

If everything is working properly, building your Rust application as a
unikernel image is as easy as running `cargo rumpbake`. Be aware though that
at the time of writing, some crates on [crates.io](https://crates.io) don't
have proper NetBSD support yet, and might will fail to compile or execute.

*Congratulations!* You are now set up to build your own Rust applications into
Rumprun unikernel images.
