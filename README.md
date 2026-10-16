# AppleHDA back in Tahoe

AppleHDA.kext has been removed starting with macOS Tahoe beta 2 and is unlikely to be included in future versions. AppleHDA.kext provides sound support to the operating system. It is required for integrated audio to work with AppleALC.kext.

It has an important role in the interaction between macOS and the sound device:

- Enables macOS to detect sound devices built into the hardware (e.g., speakers or headphones).
- Reads the Layout-ID value from the firmware, which is a unique identifier for the sound configuration of each specific Mac model.
- Based on the Layout-ID, loads sound maps specific to each Mac.

In older versions of macOS, ways were developed to patch AppleHDA.kext to make sound work on chips not supported by Apple. More recently, tools have been developed to inject sound support through the kernel, avoiding the need to modify AppleHDA.kext directly. The most commonly used tool is AppleALC.kext, created by the OpenCore team (vit9696 is the original author), which allows us to have native audio in macOS with codecs not supported by Apple without modifying the file system. It works next to AppleHDA.kext. For this reason, integrated audio is lost when AppleHDA.kext doesn't exist.

Last Intel Macs have a T2 chip that handles audio features and do not need AppleHDA.kext. Since all Mac models that can be upgraded to macOS Tahoe have a T2 chip, Apple has acted in its usual manner, removing elements that are not required by supported models.

Without AppleHDA.kext or AppleALC.kext, the ability to use digital audio through the graphics card outputs (HDMI or DisplayPort) is retained because this system relies on other kexts. But those who don't have this option lose audio through the motherboard chip since Tahoe Beta 2.

### Alternative Solution: USB Audio Chips

Many audio devices that connect to a USB port work natively on macOS; they don't require AppleHDA. I've tried Sound Blaster Play! 3, currently priced at around €15, and it works perfectly on the beta versions of macOS Tahoe. Of course, it also works on the previous macOS versions I've tested it on: Sequoia, Sonoma, and Ventura.

Disadvantages:

- Slightly higher CPU usage than the motherboard chip.
- Only two connections: speakers (headphones) and microphone, sufficient for most users, but not all.

### Reinstall AppleHDA.kext: MyKextInstaller, SimpleHDA and HDAUniversal

Developers have been looking for a way to reinstall AppleHDA.kext, extracted from Tahoe Beta 1 or Sequoia installation. It's not a simple task since this kext must be located in /System/Library/Extensions, which by design is a non-writable folder.

[OCLP](https://github.com/dortania/OpenCore-Legacy-Patcher) team is working on a patch that will be applied similarly to current root patches. Still in a beta stage, it's not yet viable for end users.

*Mirone* has [MyKextInstaller](https://github.com/Mirone/MyKextInstaller). Although it's on GitHub, source code isn't yet available. The interface is simple and user-friendly, making it very easy to use. As an added benefit, you can also download AppleHDA.kext along with the app. It modifies the system volume and rebuild kernel collections.

*Laobamac* has [SimpleHDA](https://github.com/laobamac/SimpleHDA). The source code is available. SimpleHDA is a Lilu plug-in that restores the original AppleHDA stack through OpenCore injection, allowing AppleALC to use your existing audio configuration. It does not modify the system volume or rebuild kernel collections.

*Mald0n* has [HDAUniversal](https://www.insanelymac.com/forum/topic/362932-hdauniversal-applehda-like-audio-kext-for-macos-tahoe-and-hackintosh-systems/), a modern AppleHDA-like audio kernel extension for macOS and Hackintosh systems. 

These applications allows you:

- get back sound from the built-in audio hardware by reinstalling AppleHDA.kext (extracted from first Tahoe beta) or a different audio kext.

MyKextInstaller also allows you:

- restore the system to the last unmodified system image (last sealed snapshot) to undo the process; this is not necessary with SimpleHDA or HDAUniversal.

### Requirements for MyKextInstaller

- Tahoe Kernel Debug Kit. If the same macOS version exists (e.g., `Kernel Debug Kit 26.2 build 25C56` for `macOS Tahoe 26.2 build 25C56`), that's the one to use. If it doesn't exist because Apple hasn't released it yet, use the closest previous version.
- AppleALC version 1.9.5 or later in the OpenCore Kexts folder and in config.plist (as usual)
- config.plist: csr-active-config=03080000 (SIP partially disabled)
- AppleHDA.kext extracted from the KDK. The default path is
`/Library/Developer/KDKs/<KDK_version-and-build.kdk>/System/Library/Extensions/AppleHDA.kext`

**Note**: AppleHDA.kext that exists in the macOS 15 Sequoia installation also works.

### SimpleHDA installation

- Remove other AppleHDA restoration solutions or root patches before installation to avoid loading the stack twice
- Download the RELEASE package from the [releases page](https://github.com/laobamac/SimpleHDA/releases), extract it and copy SimpleHDA.kext to EFI/OC/Kexts
- Add it to Kernel → Add in this order: Lilu → AppleALC → SimpleHDA. The package includes an example entry in OpenCore-Kernel-Add.plist
- Add `keepsyms=1` and retain the working AppleALC, layout and ACPI configuration for your machine
- Optionally install `SimpleHDA-HAL-1.0.0.pkg`
- Reboot.

There is a boot arg `shda-lang=en` that overrides the automatically detected language of device names (default Chinese). Languages supported by `shda-lang` are the same of `AppleHDA.kext`.

For any other questions, consult the README on GitHub.

### HDAUniversal installation

- Remove other AppleHDA restoration solutions or root patches before installation to avoid issues
- Run the PKG installer, available inside the downloaded ZIP
- Open `System Settings` > `Privacy & Security` and allow the HDAUniversal kernel extension if macOS asks for permission.

### MyKextInstaller screenshot

Simple interface. The app is now signed and notarized, so you will no longer see 'Move the application to the trash' message.

![MKI](Img/MKI.png)
[[title](https://github.com/dortania/OpenCore-Legacy-Patcher)](https://github.com/dortania/OpenCore-Legacy-Patcher)
### Reminder

If using MyKextInstaller, upgrading or reinstalling macOS Tahoe will lose these changes and you'll need to reinstall AppleHDA.kext again.
