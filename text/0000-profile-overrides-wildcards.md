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

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

```toml
# the `image` crate and all it's depenencies will be compiled with -Copt-level=3
[profile.dev.overrides.image."*"]
opt-level = 3
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

So given that the `gltf` crate has a dependency on `image`, it's `image` and potentially all other referenced `image` crates should be compiled with `opt-level = 3` while the rest of the `gltf` crate should be compiled with `opt-level = 2`.

# Drawbacks
[drawbacks]: #drawbacks

This complicates the build rules and cargo.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

- One alternative would be to make the syntax `[profile.dev.overrides.image]` match the `image` crate and all of it's dependencies.
- Usually when running debug builds you don't care about the debuggability of your dependant crates, but right now the only way to state that intent as it stands per RFC 2282 the only way to control this is to either opt to build all of your dependencies with the same settings and then opt-out for the ones you'd like to debug or to individually opt in to most dependant crates (and their dependencies ... ad-infinitium). Both aren't really ergonomic to use but at least offer the possibility to do specific tweaks.


# Unresolved questions
[unresolved-questions]: #unresolved-questions

- Should the proposed syntax match the crate and all it's dependencies, or should it only match the dependencies of the crate. The proposal now matches the crate and it's dependencies.
- Should we support a full path syntax such that you can tweak dependencies of crates easily? For example: `[profile.dev.override.gltf.image.png.inflate]`. This seems to be an intuitive extension; but it's counter to how cargo works & flattens it's dependencies.