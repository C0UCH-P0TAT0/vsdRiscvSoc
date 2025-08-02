## Table of Contents

| Task      							| Title                         |
|---------------------------------------------------------------|-------------------------------|
| [TASK-0](#task-0)         					| Update APT                    |
| [TASK-1](#task-1) 						| Install Development Tools     |
| [TASK-2](#task-2) 						| Create Root Directory         |
| [TASK-3](#task-3) 						| Install RISC-V Toolchain      |
| [TASK-4](#task-4) 						| Add Toolchain to PATH         |
| [TASK-5](#task-5)     					| Install DTC                   |
| [TASK-6](#task-6)   						| Install Spike                 |
| [TASK-7](#task-7)      					| Install PK                    |
| [TASK-8](#task-8)				 		| Add Spike & PK to PATH        |
| [TASK-10](#task-10)     					| Sanity Check                  |
| [FINAL DELIVERABLE](#final-deliverable) 			| Unique Test Program           |
| [TASK-9](#task-9)			 			| Install Icarus Verilog        |
| [COMMON ERRORS](#common-errors)      				| Troubleshooting               |



## TASK-0 
__UPDATE APT__

```bash
sudo apt-get update
```


## TASK-1 
__INSTALLING DEV TOOLS__

```bash
sudo apt-get install -y git vim autoconf automake autotools-dev curl \
libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave
```

## TASK-2 
__CREATING ROOT DIRECTORY__

```bash
cd
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
```

## TASK-3 
__INSTALLING RISC-V TOOLCHAIN__
__TOOLCHAIN INCLUDES CROSS-COMPILER, ASSEMBLER, LINKER, DEBUGGER etc.__

```bash
wget "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz"
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
```
 
## TASK-4 
__ADDING RISCV64 CROSS COMPILER TO PATH__
__ALLOWS LINUX TO ACCESS CROSS-COMPILER__

```bash
echo 'export PATH="$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## TASK-5 
__INSTALLING DTC__

```bash
sudo apt-get install -y device-tree-compiler
```

## TASK-6 
__INSTALLING SPIKE__
__SPIKE IS THE OFFICIAL RISCV ISA SIMULATOR (SIMULATES RISCV CPU)__

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
```

## TASK-7 
__INSTALLING PK__
__PK PROVIDES (MINIMUM) OS-LIKE FUNCTIONALITY NEEDED BY SPIKE__

```bash 
cd ../..
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
git checkout v1.0.0 
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
cd $pwd/riscv_toolchain
```

## TASK-8 
__ADDING SPIKE AND PK TO PATH__
__ALLOWS LINUX TO ACCESS SPIKE AND PK__

```bash
echo 'export PATH="$HOME/riscv_toolchain/riscv-isa-sim/build:$PATH"' >> ~/.bashrc
echo 'export PATH="$HOME/riscv_toolchain/riscv-pk/build:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## TASK-10 
__SANITY CHECK__

```bash
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
which spike
spike --version || spike -h
which pk
```

## FINAL DELIVERABLE

__CREATE UNIQUE TEST__

```bash
nano unique_test.c
```

```bash
#include <stdint.h>
#include <stdio.h>
#ifndef USERNAME
#define USERNAME "unknown_user"
#endif
#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif
// 64-bit FNV-1a
static uint64_t fnv1a64(const char *s) {
const uint64_t FNV_OFFSET = 1469598103934665603ULL;
const uint64_t FNV_PRIME = 1099511628211ULL;
uint64_t h = FNV_OFFSET;
for (const unsigned char *p = (const unsigned char*)s; *p; ++p) {
h ^= (uint64_t)(*p);
h *= FNV_PRIME;
}
return h;
}
int main(void) {
const char *user = USERNAME;
const char *host = HOSTNAME;
char buf[256];
int n = snprintf(buf, sizeof(buf), "%s@%s", user, host);
if (n <= 0) return 1;
uint64_t uid = fnv1a64(buf);
printf("RISC-V Uniqueness Check\n");
printf("User: %s\n", user);
printf("Host: %s\n", host);
printf("UniqueID: 0x%016llx\n", (unsigned long long)uid);
#ifdef __VERSION__
unsigned long long vlen = (unsigned long long)sizeof(__VERSION__) -
1;
printf("GCC_VLEN: %llu\n", vlen);
#endif
return 0;
}
```

__COMPILE__

```bash
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
-DUSERNAME="\"$(id -un)\"" -DHOSTNAME="\"$(hostname -s)\"" \
unique_test.c -o unique_test
```
__RUN__

```bash
spike ~/riscv_toolchain/riscv-pk/build/pk ./unique_test
```
__EXPECTED OUTPUT FORMAT__

```bash
RISC-V Uniqueness Check
User: <your-username>
Host: <your-hostname>
UniqueID: 0x<64-bit-hex>
GCC_VLEN: <number>
```

## TASK-9 
__INSTALLING ICARUS VERILOG__

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
git checkout --track -b v10-branch origin/v10-branch
git pull
chmod +x autoconf.sh
./autoconf.sh
./configure
make -j$(nproc)
sudo make install
```

## COMMON ERRORS

Malformed PATH: Export native compiler as path, then edit bashrc file to make sure there aren't any duplicate or erroneus path exports.

PK git was pointing to a recent version unsupported by our older toolchain. So, we degraded our PK version to fit the toolchain.


