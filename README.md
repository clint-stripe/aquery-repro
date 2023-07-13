# Reproducing bad aquery output

```
~/stripe/bazel/bazel-bin/src/bazel-dev build //:foo &&
    ~/stripe/bazel/bazel-bin/src/bazel-dev \
    aquery --skyframe_state 'outputs(".*")' --output=proto > /tmp/aquery.bin
```

textproto output consistently outputs these two messages at the end:

```
artifacts {
id: 2
path_fragment_id: 6
}
actions {
target_id: 1
action_key: "d77da09c3caf75706716060460c6c5148d163533aafffc02c47f217134f3f858"
mnemonic: "Genrule"
configuration_id: 1
arguments: "/bin/bash"
arguments: "-c"
arguments: "source external/bazel_tools/tools/genrule/genrule-setup.sh; \n        echo a > bazel-out/darwin_arm64-fastbuild/bin/a\n    "
input_dep_set_ids: 1
output_ids: 2
primary_output_id: 2
execution_platform: "@local_config_platform//:host"
}
```

## Good commit: `3ab8a0a5d628a0d958fb2eb1c0d5bb76b442e2f2`

`protoscope /tmp/aquery.bin` ends with:

```
1: {
  1: 2
  2: 6
}
2: {
  1: 1
  3: {"d77da09c3caf75706716060460c6c5148d163533aafffc02c47f217134f3f858"}
  4: {"Genrule"}
  5: 1
  6: {"/bin/bash"}
  6: {"-c"}
  6: {
    "source external/bazel_tools/tools/genrule/genrule-setup.sh; \n        echo a > ba"
    "zel-out/darwin_arm64-fastbuild/bin/a\n    "
  }
  8: {`01`}
  9: {`02`}
  13: 2
  14: {"@local_config_platform//:host"}
}
```


## Bad commit: `a2f013ac144db845e13830cd82c4acfe2455ae16`

aquery has the same textproto, but `--output=proto` ends with a partial message:

```
1: {
  1: 2
  2: 6
}
`12800208011a40643737646130396333636166373537303637313630363034363063366335313438`
`64313633353333616166666663303263343766323137313334663366383538220747656e72756c65`
`280132092f62696e2f6261736832022d633279736f757263652065787465726e616c2f62617a656c`
`5f746f6f6c732f746f6f6c732f67656e72756c652f67656e72756c652d73657475702e73683b200a`
`20202020202020206563686f2061203e2062617a656c2d6f75742f64617277696e5f61726d36342d`
`666173746275696c642f62696e2f610a`
```

<img alt="diff-screenshot" src="https://github.com/clint-stripe/aquery-repro/assets/60013602/70989f49-9f6e-492d-95b0-0dbc3ce9e4c8">
