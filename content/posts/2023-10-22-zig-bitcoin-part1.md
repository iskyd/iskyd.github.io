+++
title = 'Learning Zig and Bitcoin - Part 1'
date = 2023-10-22
+++

During this summer holidays I decided I want to learn both Zig and Bitcoin. I'm starting this series of posts both to talk about my path in learning Zig and Bitcoin, share my impressions and my knowledge and talk about the project I'm building.

#### What is Zig?

Zig is a general purpose programming language that is intended to be a replacement for the C programming language.
The main features I love about Zig are: explicit compile-time evaluation, no macros, no hidden control flows, errors management, no hidden allocations, cross compiling toolchain and inversion of control for memory allocations. Also note that Zig can compile and import C code.
To learn more about Zig I suggest you to watch [this video](https://piped.kavin.rocks/watch?v=Gv2I7qTux7g) by Andrew Kelly, the creator of Zig.


#### Let's Go

When I started learning Zig I just have basic C knowledge, basic knowledge about memory management and low level programming. I have a strong background with OOP languages that do not require memory management like PHP, Python and Julia (Julia it's not OOP but do not require explicit memory allocation and have a garbage collectors).

I started with the official documentation provided by Zig: I find the docs very clear even if it's not complete and misses some topics.

Next I started with [ziglings](https://github.com/ratfactor/ziglings), an amazing open source project that makes you learn Zig by fixing tiny broken programs.

I also started reading some books about C programming language like "The C programming language" even if I didnt' finish it yet.

#### What about Bitcoin?

At the moment I'm working for almost a year as Senior Backend Engineer in a Bitcoin company, so even if it's not my main role I got a high level knowledge of Bitcoin. I also read "Mastering Bitcoin" by Andreas Antonopoulos.

#### WALL-E

I wanted to learn Zig and deep dive in low level concepts behind Bitcoin. My first idea was to develop a full bitcoin node implementation in Zig. This was basically a porting from it's C++ implementation into Zig. I soon realized that this project was too big to develop it in my free time and also I find myself porting the existing C++ code into Zig, without forcing me to think about Zig idioms and best practices.

That's when I decided to create [WALL-E](https://github.com/iskyd/walle) a bitcoin wallet written in Zig. I don't have a roadmap an I think I will never have but my main goal is to write a time decay multi-sig wallet.

#### First steps

I started on the basics: generate a new mnemonic, a private key, a public key and derive childs from it. This are described in BIP39 and BIP32. 

I also have to implement some cryptographic operations such as secp256k1 (I can't make it works the Zig implementation in the standard library) and ripdem160.

This allows me to deep dive into mathematical concepts behind the security of the Bitcoin protocol. 

One interesting thing to implement was the modular exponentiation of very big numbers. I start with a naive implementation, make the exponentiation and then apply the modulus operation. That works fine for small numbers, but when you're dealing with very big numbers (such as in cryptography) you can't do cause you end up loosing lot of memory and you can't repesent that big numbers. The second attempt was to iterate n times (n = exponent) and multiply the base by itself and apply the modulus. This way you're never alloc more than a u256 (modulus is in u256). But... a u256 iterations will never end. That's it, you can't do that either for big numbers. Luckily [maths](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) provide us the solution and you can find my Zig implementation [here](https://github.com/iskyd/walle/blob/main/src/secp256k1/secp256k1.zig#L9). This really forces me to think about memory and time optimization, since I can't do that otherwise.

#### External libraries

At the time of writing Zig doesn't provide an official package manager to install external libraries.
I installed to external libraries: [base58](https://github.com/ultd/base58-zig) and [clap](https://github.com/Hejsil/zig-clap). The first one provide a base58 encoder and decoder while the second one provide a simple command line argument parsing.

In order to install this libraries I have created a lib/ directory and clone the repository as git submodule inside it.
Than I just added them inside build.zig as modules.

#### Conclusion

Even if Zig is not a simple language and the documentation is not complete I find the community really helpfull and I find the language amazing and explicit. I loved about compilation, memory allocation, optimization and so on.

The mathematical concepts behind Bitcoin are not easy but that's the best part.

I'm moving on with WALL-E with the hope to learn more and more concepts about Zig, Bitcoin and Maths. The next things I'm going to implement are BIP38 and transactions.
