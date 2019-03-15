- Feature Name: profile_overrides_wildcards
- Start Date: 2019-03-15
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

This is an extension to [RFC 2282](https://github.com/rust-lang/rfcs/pull/2282) to allow for more flexible wildcard support and for the recursive application of build options.

# Motivation
[motivation]: #motivation

Right now the syntax for specifying an override matches one exact crate, no matter how deep it is in de deptree. This is done to be able to target specific dependencies and override their build flags. However this requires some profiling and might be to specific.

Ideally I'd see a way to override build flags for a specific crate and all it's dependencies, as a more ergonomic feature.

When playing around with the feature as it stands now, the proposed syntax was one that I tried naturally so it feels like a logical extension.

To be able to do this cleanly we also need to support full paths in the overrides syntax.


# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation


```toml
# the `image` crate and all it's depenencies will be compiled with -Copt-level=3
[profile.dev.overrides.image."*"]
opt-level = 3

# the `inflate` create in the bottom of the `gltf` 
[profile.dev.overrides.gltf.image.png.inflate]
opt-level = 3

[profile.dev.overrides.gltf.image.'*']
opt-level = 0
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

The wording for overlapping rules as in RFC 2282 needs to be extended or changed such that weaker wild-cards or non-wildcards need to be given higher priority over broad sweeping wild-cards. For example

```toml
[profile.dev.overrides."*"]
opt-level = 0

[profile.dev.overrides.gltf."*"]
opt-level = 2

[profile.dev.overrides.image."*"]
opt-level = 3
```



# Drawbacks
[drawbacks]: #drawbacks

This complicates the build rules and cargo.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- One alternative would be to make the syntax `[profile.dev.overrides.image]` match the `image` crate and all of it's dependencies.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Should the proposed syntax match the crate and all it's dependencies, or should it only match the dependencies of the crate. The proposal now matches the crate and it's dependencies.
- How should we handle the following case: