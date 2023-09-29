This repo serves to store the testing set up used for my paper: Input Output Grammar Coverage in Fuzzing

There are git submodules for:
- Knot DNS (modified for end-to-end fuzzing with accurate code coverage)
- LibAFL (modified to implement my feedback algorithms and support feeding data to Knot)
- my custom binary parser and "vectorizer" (currently named "AndrewFuzz")

My fuzzing configuration are in a variety of branches in the LibAFL/fuzzers/forkserver_simple directory which has my custom parser as a dependency and contans a symlink to the Knot DNS executable (knot/tests-fuzz/knotd-stdio).

Instructions for running on Ubuntu:
```bash
# install old version of Rust
rustup show
rustup install 1.70.0
rustup show

# get AFLplusplus
pushd LibAFL_personal/fuzzers/forkserver_simple
cargo +1.70.0 build
popd

# build Knot
pushd knot
autoreconf -if
CC=$(realpath ../LibAFL
_personal/fuzzers/forkserver_simple/AFLplusplus/afl-clang-fast) ./configure
make check
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
