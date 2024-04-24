+++
title = 'Learning Zig and Bitcoin - Part 2 (BIP 38)'
date = 2024-02-01
+++

#### BIP 38

BIP 38 propose a method for encrypting and encoding a passphrase protected Bitcoin private key. This method use scrypt to resist brute force attack.
This method propose two encoding metodologies, but I'm going to focus only on the one that permit any private key to be encrypted with any passphrase without using the EC multiplication.
You can read more about [BIP 38 here](https://en.bitcoin.it/wiki/BIP_0038) in particular you can find the steps of what I've implemented [here](https://en.bitcoin.it/wiki/BIP_0038#Encryption_when_EC_multiply_flag_is_not_used)


#### My implementation

As you can see the steps to encrypt and decrypt a private key without EC multiplication are straighforward, but in Zig...
This implementation forces me to dive deep in the crypto code in the std lib and well, it's a crap. I've started looking in the crypto lib when using secp256k1 (prev article), but I didn't find a way to get it working so I [wrote my implementation of secp256k1](https://github.com/iskyd/walle/blob/main/src/secp256k1/secp256k1.zig).
This time I've worked with aes and scrypt in order to implement bip38 specs. The code of both lib is not clear and tests are not complete. I want also to point out that at the time of writing the crypto code inside the zig std lib has not been audited by a security expert. [There will be an audit before 1.0 release](
https://github.com/ziglang/zig/issues/5763).
You can find my implementation of Bip 38 [here](https://github.com/iskyd/walle/blob/main/src/bip38/bip38.zig). As of today there is a big major issue in my code: it doesn't return the same result as other implementation I've tried.

#### build.zig
During the development of bip 38 I needed to run tests a lot of times. Using zig build test I can execute all the tests, but I just want to run the tests specified in some files. Apparently there is no way to specify the files as args using zig build test. I can run single test using zig test commmand, but I have to link everything manually, so this would have been result in a Makefile, which I really don't want :)
Here's come the power of the zig build system: you can implement yourself this behaviour using the build.zig in few lines of code.
I have a unit_test.zig file that is the default one used by zig build test that includes all the tests. Otherwise you can specify as args the files that contains the tests. This way everything is declared inside the build.zig file and you can just run ```zig build test -- src/bip38/bip38.zig``` in order to execute only the bip 38 tests. [Here](https://github.com/iskyd/walle/blob/main/build.zig) you can find my implementation of build.zig.



#### Next steps

I need to fix the Bip 38 implementation and try to compare results with other implementation to garantuee the usability.
Next I'm going to implement transactions and transaction signature, in particular I want to implement a multisig transaction where m of n keys are required to move the funds.
Also I'm going to try to make walle usable via cli so that anyone can try it.

#### Conclusion

After few months working with Zig I can say that I'm enjoying it very much and I can't think of getting back from it. On the other side Zig seems far from releasing a 1.0 version and it lacks lot of feature/tests in particular in the crypto lib (the one that is impacting the most my project).

During this days [Zig Software Foundation announced a fundraise](https://ziglang.org/news/2024-financials/#2024-financial-report-and-fundraiser), so if you like Zig consider donating to the ZSF so that we can get closer to the 1.0.

Also [SYCL](https://sycl.it/) annouced a new event in Milan, see you there :)
