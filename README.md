This repo serves to store the testing set up used for my paper: Input Output Grammar Coverage in Fuzzing

There are git submodules for:
- Knot DNS (modified for end-to-end fuzzing with accurate code coverage)
- LibAFL (modified to implement my feedback algorithms and support feeding data to Knot)
- my custom binary parser and "vectorizer" ("combinator_fuzzer")

My fuzzing configuration are in a variety of branches in the LibAFL/fuzzers/forkserver_simple directory which has my custom parser as a dependency and contans a symlink to the Knot DNS executable (knot/tests-fuzz/knotd-stdio).

```bash
git clone git@github.com:Andrew-Fryer/MILCOM-2023-paper-setup.git
cd MILCOM-2023-paper-setup/
git submodule init
git submodule update
```

Optional: Instructions for setting up Docker container:
```bash
sudo docker run -v $(pwd):/MILCOM-2023-paper-setup -ti --rm ubuntu /bin/bash
```
...and then inside the docker container:
```bash
cd MILCOM-2023-paper-setup/
apt update
apt install curl # and accept
curl https://sh.rustup.rs -sSf | sh # and accept defaults
source "$HOME/.cargo/env"
apt install build-essential # and accept
apt-get install autoconf # and accept
apt-get install libtool autoconf automake make pkg-config liburcu-dev libgnutls28-dev libedit-dev liblmdb-dev # and accept
add-apt-repository ppa:maxmind/ppa
apt-get install libmaxminddb0
```

Instructions for running on Ubuntu or inside Docker container:
```bash
# install old version of Rust
rustup show
rustup install 1.70.0
rustup show

# get AFLplusplus
pushd LibAFL_personal/fuzzers/forkserver_simple
cargo +1.70.0 build # this will fail due to conditional compilation flags; don't worry, we just want the side effect of building AFLplusplus for us
popd

# build Knot
pushd knot
autoreconf -if
CC=$(realpath -s ../LibAFL_personal/fuzzers/forkserver_simple/AFLplusplus/afl-clang-fast) ./configure --disable-shared --disable-utilities --disable-documentation
make # this hangs sometimes...? (If you have problems building Knot, you could try just using knotd_stdio.compiled_on_ubuntu_22)
pushd tests-fuzz
make check-compile # this will create knotd_stdio
popd
popd

# build and run each fuzzer
pushd LibAFL_personal/fuzzers/forkserver_simple
ln -s ../../../knot/tests-fuzz/knotd_stdio knotd_stdio
touch coverage.csv
# git checkout different branches to select different configurations
# some configurations are also done using Rust's conditional compilation flags
./benchmark_feedbacks.sh
popd
```
