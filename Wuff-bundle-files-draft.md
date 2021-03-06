# Handling of MANIFEST.MF and plugin.xml in Wuff

Intro: Wuff is a gradle plugin for developing and assembling OSGi/Eclipse applications. Sources are available at: https://github.com/akhikhl/wuff.

This draft describes possible solution for actual problems with handling of MANIFEST.MF and plugin.xml in Wuff.

All points are open for discussion and are subject to change.

Please contribute via pull requests, issue tracker or at the mailing list: https://groups.google.com/forum/?hl=en#!forum/wuff-dev

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

- "static" and "generation" modes have the same file layout, but different handling of root bundle files.

- "merge" and "generation" modes have different file layouts, but same handling of root bundle files.

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

- Legacy projects will use static mode, because they have their own MANIFEST.MF and plugin.xml files.
- Most new projects will use generation mode. This brings reduced maintenance on MANIFEST.MF and plugin.xml.
- Some new projects will use merge mode, when wuff-generated bundle files are not sufficient.

## Build cycle

We must define how the described modes integrate into build cycle.

"Static" mode does not require special build cycle. User maintains the bundle files and invokes "build" task when needed.

"Merge" and "generation" modes require special build cycle. In particular, there should be point of time, when bundle files are merged or generated. The simplest variant would be: to introduce "processBundleFiles" task, featuring:
- "processBundleFiles" task checks whether wuff.generateBundleFiles is set to true. If not, the task does nothing.
- "classes" task depends on "processBundleFiles" task, so that builds are always done against "fresh" generated/merged files.
- it will be possible to simply invoke "processBundleFiles" from command line or in IDE.
- in a future we might integrate "processBundleFiles" task with IDE-specific gradle integration plugins, so that they invoke "processBundleFiles" whenever its inputs change.

Your suggestions and critique are welcome.
