This repo serves to store the testing set up used for my paper: Input Output Grammar Coverage in Fuzzing

There are git submodules for:
- Knot DNS (modified for end-to-end fuzzing with accurate code coverage)
- LibAFL (modified to implement my feedback algorithms and support feeding data to Knot)
- my custom binary parser and "vectorizer"

My fuzzing code is in a variety of branch in the LibAFL/fuzzers/forkserver_simple directory which has my custom parser as a dependency and contans a symlink to the Knot DNS executable (knot/tests-fuzz/knotd-stdio).
