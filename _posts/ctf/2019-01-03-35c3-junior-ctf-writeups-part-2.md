---
layout: post
title:  "35c3 junior ctf writeups part 2"
date:   2019-01-03 13:00:00 +0100
last-modified: 2019-01-06 17:00:00 +0100
categories: ctf hackthissite ccc DANCEd Ultra_Secret
math: true
head-image: 
    name: "Time"
    file:   "angelique-shelley-10.jpg"
    global: local
    creator: "Angelique Shelley"
    creatorurl: "https://www.artstation.com/artwork/ZV0Xw"
    license: CC-BY-SA

---
This writeup is going to be very short because once you know the flaw all you need to solve it is some time. This writeup and others can also be found in our [CTF writeup repo](https://github.com/HackThisSite/CTF-Writeups/tree/master/2018/35C3-Junior/).

# Challenge 2: Ultra Secret
We're given the following description:
>This flag is protected by a password stored in a highly sohpisticated chain of hashes. Can you capture it nevertheless? We are certain the password consists of lowercase alphanumerical characters only.  
>nc 35.207.132.47 1337  
>[Source](https://archive.aachen.ccc.de/junior.35c3ctf.ccc.ac/uploads/ffb8d1ab6ff961419bee1cf1cddfb2e5-ultra_secret.tar)  
>Difficulty estimate: Easy

The provided tar file contains 3 files, **```Cargo.lock```**, **```Cargo.toml```** and **```src/main.rs```**. Even though I've never done any rust it is pretty clear that [Cargo](https://doc.rust-lang.org/stable/cargo/) files are for their package manager, and the **```main.rs```** file contains the actual code we're interested in.
{% highlight rust %}{% raw %}
extern crate crypto;

use std::io;
use std::io::BufRead;
use std::process::exit;
use std::io::BufReader;
use std::io::Read;
use std::fs::File;
use std::path::Path;
use std::env;

use crypto::digest::Digest;
use crypto::sha2::Sha256;

fn main() {
    let mut password = String::new();
    let mut flag = String::new();
    let mut i = 0;
    let stdin = io::stdin();
    let hashes: Vec<String> = BufReader::new(File::open(Path::new("hashes.txt")).unwrap()).lines().map(|x| x.unwrap()).collect();
    BufReader::new(File::open(Path::new("flag.txt")).unwrap()).read_to_string(&mut flag).unwrap();

    println!("Please enter the very secret password:");
    stdin.lock().read_line(&mut password).unwrap();
    let password = &password[0..32];
    for c in password.chars() {
        let hash =  hash(c);
        if hash != hashes[i] {
            exit(1);
        }
        i += 1;
    }
    println!("{}", &flag)
}

fn hash(c: char) -> String {
    let mut hash = String::new();
    hash.push(c);
    for _ in 0..9999 {
        let mut sha = Sha256::new();
        sha.input_str(&hash);
        hash = sha.result_str();
    }
    hash
}
{% endraw %}{% endhighlight %}

The code is very short and it doesn't do much:
1. read hashes.text
2. ask user for password
3. split that password into 32 chars (better to crash than accept passwords that aren't 32 characters long I guess)
4. hash each character (10000 times) and compare it to the corresponding hash in hashes.txt

If you've absolutely never done anything security related you might mistakenly think "more hashes must be better" and "hashing more times must be better". We'll see why that is wrong in a second.

First problem of splitting the password into multiple hashes for each character means I only need to crack hashes that I know are 1 character long, it'll take no time at all. So the strength of your 32character password is now the same as one that is 1 character long. Of course I don't have these hashes (yet) to crack them, so just this isn't enough for me to easily get in.

Second problem is hashing things 10000 times. While hashing things multiple times does increase the hash time and is a valid technique to make hashes harder to crack, in combination with the first problem it is disastrous.  
We're not hashing 1 thing 10000 times, we're hashing every character 10000 times, that means 320000 [sha256](https://en.wikipedia.org/wiki/SHA-2) hashes.. that takes a very long time (about 21.4 seconds on the challenge server).

Third and fatal problem is that once the server realizes the password doesn't match it stops hashing and immediately returns. This allows us to perform a [timing attack](https://en.wikipedia.org/wiki/Timing_attack).

### Timing attack
> In cryptography, a timing attack is a side channel attack in which the attacker attempts to compromise a cryptosystem by analyzing the time taken to execute cryptographic algorithms. Every logical operation in a computer takes time to execute, and the time can differ based on the input; with precise measurements of the time for each operation, an attacker can work backwards to the input.[1]
>
>Information can leak from a system through measurement of the time it takes to respond to certain queries. How much this information can help an attacker depends on many variables: crypto system design, the CPU running the system, the algorithms used, assorted implementation details, timing attack countermeasures, the accuracy of the timing measurements, etc.
>
>Timing attacks are often overlooked in the design phase because they are so dependent on the implementation and can be introduced inadvertently with compiler optimizations. Avoidance of timing attacks involves design of constant-time functions and careful testing of the final executable code.[1] 
>
> -- [wikipedia](https://en.wikipedia.org/wiki/Timing_attack)

Timing attacks aren't just limited to cryptography related implementations like in this case. A simple string comparison in any language could make it vulnerable to a timing attack if the standard methods for string comparison are used.  
If the thing you're comparing to needs to stay a secret you should use a comparison function with a fixed length. That is, keep checking each character for a match even if you already know it is wrong.

##### Example
A really simple and dumb example would be a numerical keypad controlling a door that checks each number entered and gives you an error as soon as it finds a number that doesn't match its preprogrammed sequence.  
You'd be able to enter 1111 2111...9111 and once the first number is correct it'd take it slightly longer to respond (or worse it'd not even wait for you to enter all the numbers and gives an error as soon as you press the digit).

When guessing the number to enter the building for a 4 digit code you'd need 10000 attempts (0000-9999) in the absolute worst case.  
But if it is vulnerable to a timing attack you'd only need 40 attempts at most (10 for each digit), significantly less.

Since this example describes an object you'd have physical access to, your time measurements could be very precise. The solution for the manufacturer would be to program its device to store all the entered numbers in a single number and compare them at the same time, which wouldn't cause a time variation when comparing a wrong sequence to a slightly less wrong sequence.

##### Our attack
Our attack will work the same way as the keypad example. We'll start with a wrong password aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa, and change one character at a time until it takes longer to respond.

We could easily write a script to do this, but in this case it is so simple it might be easier to just do it manually than bother writing something.

We'll use the time command to measure the time it takes for the server to close the connection, and while we're doing it manually, we can still do it in bulk. Since we know the password is lowercase alphanumerical I simply pasted the following commands into my terminal in 1 go.

{% highlight bash %}{% raw %}
time ( echo "0aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "1aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "2aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "3aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "4aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "5aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "6aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "7aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "8aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "9aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "baaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "caaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "daaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "eaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "faaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "gaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "haaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "iaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "jaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "kaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "laaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "maaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "naaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "oaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "paaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "qaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "raaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "saaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "taaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "uaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "vaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "waaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "yaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
time ( echo "zaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" | nc 35.207.132.47 1337)
{% endraw %}{% endhighlight %}
(NOTE: the challenge server might not be available or change in the future, please check the [archive](https://archive.aachen.ccc.de/junior.35c3ctf.ccc.ac/))

This will run each of those commands sequentially and show us how long it took to complete. We'd be able to see that 1aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa took a bit longer to return than all the others and thats how we know the first character was a 1.

Next we try the next sequence 10aaaaaaaa...aa to 1zaaaaaaa...aa and pick the longest of those. As we get more and more correct characters it'll take significantly longer per attempt. So if you have the screen real-estate like I do just run it somewhere in view while you do other things and interupt the commands (CTRL+C) when you see you notice you got a hit. Interupting when you get a hit will make it a lot faster than trying all variations and checking which character took the longest after doing it all (also, I noticed pretty quickly that the password was more like 0-9,a-f, not a-z, so even worse case became less horrible).

I don't know how long it took to perform the attack, but the attempts to find the first character of the password only took 0.7 seconds, the last character took 21.4 seconds per attempt.

In the end I was greeted with the following:

{% highlight bash %}{% raw %}
weekend@haxtop:~$ time ( echo "10e004c2e186b4d280fad7f36e779ed4" | nc 35.207.132.47 1337)
Please enter the very secret password:
35C3_timing_attacks_are_fun!_:)
{% endraw %}{% endhighlight %}

### Conclusion
###### Use proper hashes
Pretty similar to the previous writeup, **don't** write your own weird hashing implementations if you don't know what you're doing. 

If they just hashed the password once using sha256 it would have taken a very long time to crack that password.  
In fact lets look at how long exactly. 

If we assume the characters were lowercase alphanumerical, we'd have 36 possible characters (0-9,a-z). There are 32 characters.
That would result in $$36^{32}$$ possible combinations. Let's assume we got a nice computer capable of doing 1000000 hashes per second:  
$${% raw %} 36^{32} = 6.334028666\times10^{49} \text{ combinations} \\
36^{32} \text{ combinations} \quad/\quad 1000000 \text{ hashes/sec } = 6.334028666\times10^{43} \text{ seconds} \\
6.334028666\times10^{43} \text{ seconds} \quad/\quad 60 \text{ seconds} = 1.055671444\times10^{42} \text{ minutes} \\
1.055671444\times10^{42} \text{ minutes} \quad/\quad 60 \text{ minutes} = 1.759452407\times10^{40} \text{ hours} \\
1.759452407\times10^{40} \text{ hours} \quad/\quad 24 \text{ hours} = 7.331051694\times10^{38} \text{ days} \\
7.331051694\times10^{38} \text{ days} \quad/\quad 360 \text{ days} = 2.036403248\times10^{36} \text{ years} \\
{% endraw %}$$  
Even if you have a computer 10, 100 or 1000 times faster than the example 1 million hashes per second it'd still take way too long. Compared to the $$36\times32=1152$$ attempts max it'd take to crack these hashes.

Timing attacks aren't hard to prevent once you know how they work. So just remember this challenge and don't make these kinds of silly mistakes.

###### Rust seems okay
The final conclusion is that rust seems nice but I still have an irrational dislike towards package managers for languages. The **```Cargo.lock```** file was almost 3 times as long as our 45 lines of actual code because it has a single dependency (which had several dependencies of its own, and they had some, etc etc). Also how does version ```0.3.22``` of the ```rand``` package depend on version ```0.4.3``` of that same package?

No point to keep complaining about this way of coding though, all languages seem to be going this way despite all the [problems](https://arstechnica.com/information-technology/2017/09/devs-unknowingly-use-malicious-modules-put-into-official-python-repository/) it [repeatedly](https://www.zdnet.com/article/twelve-malicious-python-libraries-found-and-removed-from-pypi/) [causes](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/).
