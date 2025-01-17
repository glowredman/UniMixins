# Developer usage

An example mod showcasing the mod's various features is available in the `example-*` branches of the repo:
- [`example-fg-g4`](https://github.com/LegacyModdingMC/UniMixins/tree/example-fg-g4) - ForgeGradle-based build script with Gradle 4
- [`example-fg-g6`](https://github.com/LegacyModdingMC/UniMixins/tree/example-fg-g6) - ForgeGradle-based build script with Gradle 6

Below are instructions for migrating build scripts to use UniMixins.

## RetroFuturaGradle

The easiest way to use UniMixins is to use [GTNH's ExampleMod](https://github.com/GTNewHorizons/ExampleMod1.7.10), which has UniMixins integration built-in.

## ForgeGradle

Refer to the example mod's build script ([Gradle 4](https://github.com/LegacyModdingMC/UniMixins/blob/example-fg-g4/build.gradle#L79) | [Gradle 6](https://github.com/LegacyModdingMC/UniMixins/blob/example-fg-g6/build.gradle#L91)).

You can also depend on modules individually:

```gradle
dependencies {
    // One of these (note: this module doesn't have dev jars)
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixin-1.7.10:$unimixinsVersion+unimix.0.12.2-mixin.0.8.5")
    //implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixin-1.7.10:$unimixinsVersion+spongepowered.0.8.5")
    //implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixin-1.7.10:$unimixinsVersion+fabric.0.12.5-mixin.0.8.5")
    //implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixin-1.7.10:$unimixinsVersion+gasmix.0.8.5-gasstation_7")
    //implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixin-1.7.10:$unimixinsVersion+gtnh.0.8.5-GTNH-2")
    
    // You don't need all the modules, only the ones your mod requires.
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-compat-1.7.10:$unimixinsVersion:dev")
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-spongemixins-1.7.10:$unimixinsVersion+gtnh.2.0.1:dev")
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixinbooterlegacy-1.7.10:$unimixinsVersion+1.2.1:dev")
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-gasstation-1.7.10:$unimixinsVersion+0.5.1:dev")
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixinextras-1.7.10:$unimixinsVersion+0.2.0:dev")
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-gtnhmixins-1.7.10:$unimixinsVersion+2.1.15:dev")
    implementation("com.github.LegacyModdingMC.UniMixins:unimixins-mixingasm-1.7.10:$unimixinsVersion+0.3:dev")
}
```

> If you are *not* using UniMix or GTNH's Mixin fork, you will need to add the compat module and set the `-Dunimixins.compat.hackClasspathModDiscovery=true` JVM flag, or Forge may fail to discover your mod or its dependencies.

## Tricks

### Hot swapping

Mixin hot swapping can be enabled in a similar way to how it works on [Fabric](https://fabricmc.net/wiki/tutorial:mixin_hotswaps). You just have to add the UniMixins jar (or the matching UniMix jar) as a java agent.

Here's a buildscript snippet that will set it up for you (tested with the GTNH ExampleMod):

```gradle
afterEvaluate {
    File uni = configurations.compileClasspath.findAll { it.name.contains("unimixins-all-") || it.name.contains("unimixins-mixin-") || it.name.contains("unimixins-0.") }.first()
    runClient {
        extraJvmArgs.add(
                '-javaagent:' + uni.getPath()
        )
    }
}
```

## Pitfalls

### Shaded ASM package name

Mods which depend on MixinBooterLegacy or GTNHMixins use the MixinBooterLegacy-style shaded ASM package name (`org.spongepowered.libraries.org.objectweb.asm`), whereas UniMix uses the legacy name (`org.spongepowered.asm.lib`). You will have to switch to the latter to make your mod compile.

```patch
-import org.spongepowered.libraries.org.objectweb.asm.tree.ClassNode;
+import org.spongepowered.asm.lib.tree.ClassNode;
```

### `--mods` hack

Some build scripts use the `--mods` program argument to add a duplicate of the mod in order to work around Mixin [preventing mixin mods on the class path from getting loaded as Forge mods.](https://github.com/SpongePowered/Mixin/issues/207) In UniMix and GTNH's Mixin fork, this issue was fixed, so this workaround should be removed since it only causes problems.

```patch
-       arguments += [
-               "--mods=../build/libs/$archivesBaseName-${version}.jar"
-       ]
```
