# docker-spectre
A dockerized spectre test environment. This image tests for the [spectre vulnerability](https://meltdownattack.com/), also known as [CVE-2017-5753](https://www.cve.mitre.org/cgi-bin/cvename.cgi?name=2017-5753), [CVE-2017-5715](https://www.cve.mitre.org/cgi-bin/cvename.cgi?name=2017-5715) and also on [Exploit-DB:43427](https://www.exploit-db.com/exploits/43427/). Also [CVE-2017-5754](https://www.cve.mitre.org/cgi-bin/cvename.cgi?name=2017-5754) a.k.a MeltDown is included here.

## Introductionary reading / TL;DR
* Original POC used here: [Eriks GIST](https://gist.github.com/ErikAugust/724d4a969fb2c6ae1bbd7b2a9e3d4bb6)
* [spectre_multiarch: Architecture independent version](https://github.com/adrb/public/tree/master/linux/spectre_multiarch)
* [Deep learning side channel privileged memory reader](https://github.com/asm/deep_spectre)
* [Am-I-affected-by-Meltdown](https://github.com/raphaelsc/Am-I-affected-by-Meltdown)

## Prerequisites
* a system running an actual version of gcc, e.g. Ubuntu 16.04 LTS

## Build spectre POC
The POC will be compiled via gcc and containerized into a docker scratch container. Let's begin by cloning this repository:
```
git clone git@github.com:feffi/docker-spectre.git
cd docker-spectre
git submodule init
```
Then we create a base scratch image where we will bundle the binaries in for easy deployment:
```
tar cv --files-from /dev/null | docker import - scratch
```
Everything is prepared now, let's start compiling:
```
gcc -static -std=c99 -O0 spectre.c -o spectre
```
We can now bundle the binary in the prepared docker scratch image:
```
echo "FROM scratch\nADD spectre /spectre\nCMD ["/spectre"]\n" > Dockerfile
docker build -t docker-sprectre:latest .
```
Now you're able to run this test as a normal containerized app anywhere between your local machine or cluster scale on kubernetes.
```
docker run -ti --privileged docker-spectre:latest
```
If your system is affected, you'll be seeing this output:
```
Reading 40 bytes:
Reading at malicious_x = 0xffffffffffdd75a8... Success: 0x54=’T’ score=7 (second best: 0x0B score=1)
Reading at malicious_x = 0xffffffffffdd75a9... Success: 0x68=’h’ score=2
Reading at malicious_x = 0xffffffffffdd75aa... Success: 0x65=’e’ score=2
Reading at malicious_x = 0xffffffffffdd75ab... Success: 0x20=’ ’ score=2
Reading at malicious_x = 0xffffffffffdd75ac... Success: 0x4D=’M’ score=2
Reading at malicious_x = 0xffffffffffdd75ad... Success: 0x61=’a’ score=2
Reading at malicious_x = 0xffffffffffdd75ae... Success: 0x67=’g’ score=2
Reading at malicious_x = 0xffffffffffdd75af... Success: 0x69=’i’ score=2
Reading at malicious_x = 0xffffffffffdd75b0... Success: 0x63=’c’ score=2
Reading at malicious_x = 0xffffffffffdd75b1... Success: 0x20=’ ’ score=2
Reading at malicious_x = 0xffffffffffdd75b2... Success: 0x57=’W’ score=2
Reading at malicious_x = 0xffffffffffdd75b3... Success: 0x6F=’o’ score=2
Reading at malicious_x = 0xffffffffffdd75b4... Success: 0x72=’r’ score=2
Reading at malicious_x = 0xffffffffffdd75b5... Success: 0x64=’d’ score=2
Reading at malicious_x = 0xffffffffffdd75b6... Success: 0x73=’s’ score=2
Reading at malicious_x = 0xffffffffffdd75b7... Success: 0x20=’ ’ score=2
Reading at malicious_x = 0xffffffffffdd75b8... Success: 0x61=’a’ score=2
Reading at malicious_x = 0xffffffffffdd75b9... Success: 0x72=’r’ score=2
Reading at malicious_x = 0xffffffffffdd75ba... Success: 0x65=’e’ score=7 (second best: 0x0B score=1)
Reading at malicious_x = 0xffffffffffdd75bb... Success: 0x20=’ ’ score=2
Reading at malicious_x = 0xffffffffffdd75bc... Success: 0x53=’S’ score=2
Reading at malicious_x = 0xffffffffffdd75bd... Success: 0x71=’q’ score=2
Reading at malicious_x = 0xffffffffffdd75be... Success: 0x75=’u’ score=2
Reading at malicious_x = 0xffffffffffdd75bf... Success: 0x65=’e’ score=2
Reading at malicious_x = 0xffffffffffdd75c0... Success: 0x61=’a’ score=2
Reading at malicious_x = 0xffffffffffdd75c1... Success: 0x6D=’m’ score=2
Reading at malicious_x = 0xffffffffffdd75c2... Success: 0x69=’i’ score=2
Reading at malicious_x = 0xffffffffffdd75c3... Success: 0x73=’s’ score=2
Reading at malicious_x = 0xffffffffffdd75c4... Success: 0x68=’h’ score=2
Reading at malicious_x = 0xffffffffffdd75c5... Success: 0x20=’ ’ score=2
Reading at malicious_x = 0xffffffffffdd75c6... Success: 0x4F=’O’ score=2
Reading at malicious_x = 0xffffffffffdd75c7... Success: 0x73=’s’ score=2
Reading at malicious_x = 0xffffffffffdd75c8... Success: 0x73=’s’ score=2
Reading at malicious_x = 0xffffffffffdd75c9... Success: 0x69=’i’ score=2
Reading at malicious_x = 0xffffffffffdd75ca... Success: 0x66=’f’ score=2
Reading at malicious_x = 0xffffffffffdd75cb... Success: 0x72=’r’ score=2
Reading at malicious_x = 0xffffffffffdd75cc... Success: 0x61=’a’ score=2
Reading at malicious_x = 0xffffffffffdd75cd... Success: 0x67=’g’ score=2
Reading at malicious_x = 0xffffffffffdd75ce... Success: 0x65=’e’ score=2
Reading at malicious_x = 0xffffffffffdd75cf... Success: 0x2E=’.’ score=2
```

## Meltdown POC
Same procedure with the Meltdown check binary, first switch to the POCs directory ([git submodule](https://git-scm.com/book/de/v1/Git-Tools-Submodule)), then compile the binary:
```
cd ./Am-I-affected-by-Meltdown
make
```
Again, after successfull compiling, package the check into an docker container, this time a Ubuntu baseimage:
```
echo "FROM phusion/baseimage\nADD meltdown-checker /meltdown-checker\nCMD ["/meltdown-checker"]\n" > Dockerfile
docker build -t docker-meltdown:latest .
```
This test can now be run as a normal containerized app anywhere between your local machine or cluster scale on kubernetes.
```
docker run -ti --privileged docker-meltdown:latest
```
If your system is affected, you'll be seeing this output:
```
Checking whether system is affected by Variant 3: rogue data cache load (CVE-2017-5754), a.k.a MELTDOWN ...
Checking syscall table (sys_call_table) found at address 0xffffffffaea001c0 ...
0xc4c4c4c4c4c4c4c4 -> That's unknown
0xffffffffae251e10 -> That's SyS_write

System affected! Please consider upgrading your kernel to one that is patched with KAISER
Check https://security.googleblog.com/2018/01/todays-cpu-vulnerability-what-you-need.html for more details
```

## A word of warning
As of the statistical nature of this breaches, it's possible and even likely that both can produce false positives AND false negatives. To overcome this, a good recommendation would be to run the checks several times an diff the reports.

No warranty though!

## Mitigation of both POCs
### spectre
Work in progress...

### Meltdown
Patches for some major OSes are already available.
