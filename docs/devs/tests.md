---
title: Tests
---

## Tests

You can build the tests by setting the `BUILD_TESTS` flag during configuration:
```bash
cmake -B build -DBUILD_TESTS=ON
```

These include the single instruction tests from [FEX-Emu](https://github.com/FEX-Emu/FEX/tree/main/unittests/ASM), some unit tests for syscalls and some binary tests.

The felix86 CI runs more binary tests than that. If you want to run the binary tests, clone the binary_tests repo:
```bash
git clone https://github.com/felix86-emu/binary_tests
cd binary_tests
```

You can use the `install_tests.sh` script to download the binary tests:
```sh
./install_tests.sh $(pwd)/binaries
```

Then to run the tests, temporarily disable binfmt_misc usage and use the `run_tests.sh` script:
```sh
export FELIX86_BINFMT_MISC_INSTALLED=0
./run_tests.sh /path/to/felix86 $(pwd)/binaries
```
