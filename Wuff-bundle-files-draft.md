# Wuff MANIFEST.MF and plugin.xml handling

All points are open for discussion and are subject to change.

Please contribute via pull requests, issue tracker or at the discussion group: https://groups.google.com/forum/?hl=en#!forum/wuff-dev

## Current implementation

Currently Wuff processes MANIFEST.MF and plugin.xml (later called "bundle files") 
in the following ways:

- when bundle files not present in project root, then Wuff generates bundle files in buildDir. 
 
- when bundle files present in project root, then Wuff performs merge: 
`existing_bundle_files + wuff_generated_files -> merged_bundle_files`. Merged bundle files are placed in buildDir.

## Problems with current implementation

- Eclipse IDE does not see the merged bundle files (they are in buildDir).

- It's not possible to turn off bundle files merge.

## Possible solution

Let's consider the following distinct modes of MANIFEST.MF and plugin.xml handling.

### Static mode

A bundle contains user-defined bundle files in project root. Wuff does not modify them and Eclipse IDE sees them.

```
some-bundle
  META-INF
    MANIFEST.MF
  plugin.xml
```

### Merge mode

A bundle contains user-defined bundle files in src/main/bundle. Wuff performs merge: 
`existing_bundle_files + wuff_generated_files -> merged_bundle_files`. Merged bundle files are:
- placed in project root 
- always overwritten
- seen by Eclipse IDE

```
some-bundle
  src
    main
      bundle
        META-INF
          MANIFEST.MF
        plugin.xml
  META-INF
    MANIFEST.MF    # merged
  plugin.xml       # merged
```

### Generation mode

A bundle does not contain user-defined bundle files. Wuff automatically generates bundle files. Generated bundle files are:
- placed in project root 
- always overwritten
- seen by Eclipse IDE

```
some-bundle
  META-INF
    MANIFEST.MF    # generated
  plugin.xml       # generated
```

Similarities and differences between modes:

- "static" and "generation" modes has the same file layout, but different handling of root bundle files.

- "merge" and "generation" modes has different file layouts, but same handling of root bundle files.

It appears that all three modes can be controlled with single property:

```groovy
wuff {
  generateBundleFiles = true
}
```

How generateBundleFiles works:

- when generateBundleFiles is false (the default), we get static mode.

- when generateBundleFiles is true and there are user-defined bundle files in "src/main/bundle", we get merge mode.

- when generateBundleFiles is true and there are no user-defined bundle files in "src/main/bundle", we get generation mode.

How this will apply to Wuff-driven projects:

- Legacy projects will use static mode, because they use their own MANIFEST.MF and plugin.xml files.
- Most new projects will use generation mode. This brings reduced maintenance on MANIFEST.MF and plugin.xml.
- Some new projects will use merge mode, when wuff-generated bundle files are not sufficient.

Your suggestions and critique are welcome.
