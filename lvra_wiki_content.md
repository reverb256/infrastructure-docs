# LVRA Wiki - VRChat Setup, NixOS VR, and Performance Best Practices
# Fetched from wiki.vronlinux.org (formerly lvra.gitlab.io)

================================================================================
PAGE 1: VRChat Main
Source: https://lvra.gitlab.io/docs/vrchat/
================================================================================

# VRChat

The most popular social VR game, here are resources to get the best experience on Linux.

## Recommended Proton

Current recommended Proton: Proton-GE-RTSP

As opposed to the default setting, this Proton version enables more stable and feature rich video playback. Featuring a number of fixes and most prominently playback of livestreamed content, typically real time streaming protocol found at live events and often utilized by the VRCDN.

As of the GE-Proton9-10-rtsp14 release, video content should be working correctly. Please report any notable stability bugs to LVRA general chat or by creating an issue on GitHub.

This tar file can be extracted to a ~/.local/share/Steam/compatibilitytools.d/ (create if it does not already exist) folder and steam restarted to set the game Proton version to rtsp.

## Common issues

If you utilize PipeWire as your audio server, VRChat has a tendency to drop multiple seconds of audio over DisplayPort connections under load for HMDs like the Valve Index. This workaround is required not to drop audio from time to time.

Similar tuning off PulseAudio can fix the issue if encountered there.

Should the game prompt to exit the game upon an anti-cheat failure, simply try to load the game again until it works. See EAC section.

Given a few caveats most video players will work.

## Privacy

VRChat drags quite a tangle of privacy concerns.

Please consider your opsec in relation to all VR API input including poses, all voice communication, unsandboxed code execution, the EAC (anticheat) system which occasionally probes the list of all system processes, the game maintaining a memory-buffered audio/video recording of the window contents tied to the report system, and much more.

In-game analytics can be blocked at the DNS level. These are often external services the game phones into to deliver usage statistics. Opt-out by blocking this domain list in your network or computer firewall/DNS service: https://github.com/Luois45/VRChatAnalyticsBlocklist

================================================================================
PAGE 2: Easy Anti-Cheat (EAC)
Source: https://lvra.gitlab.io/docs/vrchat/eac/
================================================================================

# Easy Anti-Cheat

Running VRChat on Linux used to be prone to unwarranted EAC errors. Sometimes, seemingly randomly, you would get an EAC error during the VRChat startup/login screen. It looks like a message box, with the title "Anti-cheat Error", a message referring to a filepath that failed to validate integrity, and a single button labelled "Quit". This would trigger even when all files were unmodified.

Thankfully, at some point in October or November 2024, these EAC errors resolved themselves. While it is still possible to get EAC errors for other reasons, the race condition that made it a coin-flip on startup no longer plagues us.

If you are running into EAC errors still, try these options:

- Trigger a reinstall of the EAC runtime by moving the Proton prefix for VRChat somewhere else, forcing Steam to regenerate it on next launch
- Make sure you are not blocking the EAC domains, such as modules-cdn.eac-prod.on.epicgames.com
- Do not set SDL_VIDEODRIVER environment variable anywhere - this breaks the splash screen
- Do not use the VR_OVERRIDE environment variable

The below workaround shouldn't be necessary anymore, but is kept here in case the issue comes back someday.

## Workaround

Wrapper script for starting VRChat: startvrc.sh on GitHub

Set startup options for VRChat: /path/to/startvrc.sh %command%

If you're using any extra environment variables, they should go first:
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 /path/to/startvrc.sh %command%

================================================================================
PAGE 3: VRChat Performance
Source: https://lvra.gitlab.io/docs/vrchat/performance/
================================================================================

# Performance

VRChat is notorious for exhibiting very bad performance. Here are some tips for getting a smoother experience:

## Recommended Settings

This gives a good baseline for anyone to start with. Feel free to tweak and experiment further.

- Anti-Aliasing: Off or 2x
  - Greatly increases GPU load (even more so on Nvidia).
  - Try with off first, if the jagged edges bother you too much, try 2x.

- Pixel Light Count: Low
  - Each pixel light adds significant CPU load.
  - It's extremely common for new world creators to forget to bake lighting and upload a world with too many realtime lights, killing everyone's performance.
  - Turning these completely off will make some worlds look incorrect (too dark).

- Shadow Quality: Low
  - Performance hit can be disproportionately high for visual benefit.

- LOD Quality: Low or Medium
  - Only makes a real difference in complex scenes, such as forests, cityscapes, etc.
  - Adds GPU load based on how complex the scene is.

- Particle Limiter: On

## CPU Bottlenecks

In VRC, it's very easy to hit a CPU bottleneck. Avatar material count, avatar mesh count, avatar animators, pixel lights, phys bones, particles all contribute to CPU load.

By far the biggest contributors are unoptimized avatars and unoptimized worlds.

### How to check if you're bottlenecked by CPU or GPU

- Check GPU usage in nvtop (any GPU) or nvidia-smi (Nvidia-only).
  - If it's not at 100%, you're likely CPU bottlenecked.
- SteamVR: Lower the SteamVR render resolution.
  - If your FPS doesn't increase, you're likely CPU bottlenecked.

### What to do when CPU bottlenecked

Here are some tips to ease VRChat's CPU hit on your system.

#### Reduce Maximum Shown Avatars

I recommend 10-15 as a base setting, likely won't be interacting with more people than that at the same time.

#### Block by default, utilize Show Avatar

- Set up a custom Safety profile where you block Animators and Shaders for everyone (possibly even for friends).
- Block Poorly Optimized Avatars: Very Poor
- Manually Show Avatar people who you are actively interacting with.

This gives you full control on what you're seeing, and will be able to quickly identify that one person whose avatar is wrecking your FPS.

## Avatar Optimization

Are you running around in a Very Poor avatar? Consider this a read. (And also poke your friends to do the same!)

There are tools that let you do this in a few clicks, even if you are completely clueless about what to do. You may also want to check out VRChat's documentation on optimization tips.

### d4rkAvatarOptimizer

Install: vrc-get repo add https://d4rkc0d3r.github.io/vpm-repos/main.json

### Avatar Optimizer by Anatawa12

Install: vrc-get repo add https://vpm.anatawa12.com/vpm.json

================================================================================
PAGE 4: NixOS
Source: https://lvra.gitlab.io/docs/distros/nixos/
================================================================================

# NixOS

General documentation about VR is provided on the NixOS Wiki.

The recommended way to set up NixOS for VR is by using services.monado or services.wivrn. Please see below for specific documentation on these 2 methods.

## OpenXR

### Runtimes

To set the default OpenXR runtime, you can either use services.monado.defaultRuntime or create/modify ~/.config/openxr/1/active_runtime.json. This is not necessary for WiVRn as it handles switching the active runtime while the headset is connected.

Warning: OpenXR apps running under Steam (or more generally, under a sandbox) will not be able to find the OpenXR runtime if they cannot access openxr/1/active_runtime.json in XDG_CONFIG_HOME or any of the paths in XDG_CONFIG_DIRS (such as $HOME/.config/openxr/1/active_runtime.json, /etc/xdg/openxr/1/active_runtime.json). If using WiVRn, this is handled automatically.

You need to link the active_runtime.json so that the link target is accessible in the sandbox (make sure /nix is passed through to the container; this is already the case for Steam). This can be done through Home Manager by setting:

  xdg.configFile."openxr/1/active_runtime.json".source = "${pkgs.monado}/share/openxr/1/openxr_monado.json";

#### Monado

Monado is supported natively on NixOS using services.monado since 24.05. You may also want to see the NixOS wiki on Monado.

If you are using the Monado NixOS module, it's recommended to manage it using the provided systemd user service (systemctl --user start monado.service and systemctl --user stop monado.service).

Tip: Due to the presence of the monado.socket, Monado should also start up automatically whenever an OpenXR app attempts to connect.

If you wish for Monado to also stop when all XR apps close, set systemd.user.services.monado.environment.IPC_EXIT_WHEN_IDLE = "1". systemd.user.services.monado.environment.IPC_EXIT_WHEN_IDLE_DELAY_MS = "n" can be used to set how long it waits for a new client before stopping Monado when all apps close, where n is a number in milliseconds.

If you are having performance issues, you may want to try something below:

- If you are running a NixOS version prior to !503439, you should set U_PACING_APP_USE_MIN_FRAME_PERIOD = "1" for a large performance increase.
- In case of the headset view stuttering, adding U_PACING_COMP_MIN_TIME_MS = "5" to systemd.user.services.monado.environment could help. Adjust the value as needed.
- Similarly, setting the CPU niceness value to a higher priority manually with renice -20 -p $(pgrep monado) could also help. Unfortunately systemd.user.services.monado.serviceConfig.Nice = -20; does not seem to work.

#### WiVRn

WiVRn is also supported natively on NixOS using services.wivrn since 24.05.

As WiVRn is built around Monado, most, if not all, settings for Monado are also available for WiVRn, however you may need to change things like process and option names appropriately.

You can find WiVRn on the official NixOS wiki.

### Steam games and OpenVR apps

For running OpenVR apps (such as games intended to work with SteamVR), you may also need to install xrizer or opencomposite for OpenVR compatibility.

You are highly recommended to view Monado's page on OpenVR apps for more info.

As you need to set PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 for games to connect to the OpenXR runtime, it may be easier to set this in the Steam FHS through Nix:

  programs.steam = {
    enable = true;
    package = pkgs.steam.override {
      extraProfile = ''
        # Allows Monado/WiVRn to be used
        export PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1
        # Fixes timezones on VRChat
        unset TZ
      '';
    };
  };

As on NixOS you're probably wanting to do things declaratively, here's an example on how to set ~/.config/openvr/openvrpaths.vrpath to point to xrizer or OpenComposite using Home Manager:

  xdg.configFile."openvr/openvrpaths.vrpath".text = let
    steam = "${config.xdg.dataHome}/Steam";
  in builtins.toJSON {
    version = 1;
    jsonid = "vrpathreg";
    external_drivers = null;
    config = [ "${steam}/config" ];
    log = [ "${steam}/logs" ];
    runtime = [
      "${pkgs.xrizer}/lib/xrizer"
      # OR
      #"${pkgs.opencomposite}/lib/opencomposite"
    ];
  };

## SteamVR

SteamVR works like it does on other distros for the most part. Unfortunately, if it doesn't work out of the box, troubleshooting it on NixOS can be close to impossible due to NixOS's structure and SteamVR's proprietary nature.

Asynchronous reprojection does not work without a kernel patch. However, this patch is only applicable for AMD GPUs. There is no way to get SteamVR asynchronous reprojection working on Nvidia.

setcap doesn't work but can be done manually by running:
  sudo setcap CAP_SYS_NICE=eip ~/.local/share/Steam/steamapps/common/SteamVR/bin/linux64/vrcompositor-launcher

### WayVR

WayVR will not run under SteamVR by running it normally, instead you need to run steam-run wayvr. This is due to the Steam FHS stopping it from accessing certain things on NixOS.

## Envision

Warning: On NixOS it is highly recommended to not use Envision. Consider using the config options listed above.

Envision is packaged in nixpkgs but frequently breaks due to updates. It may also mess with your monado.service setup in unexpected ways.

Also, if you try to use Envision with a dedicated PCVR headset (i.e., Monado with an Index, Pimax, etc), asynchronous reprojection will not work without an AMD-only kernel patch, for the same reason as SteamVR.

### Removing Envision

In case you did not follow the above advice, and attempted to use Envision on NixOS, you may want to undo the changes it has made to your home folder, so services.monado, services.wivrn, opencomposite and/or xrizer may work correctly.

If you just want a 1-line command to just remove its stuff:
  rm ~/.config/openxr/1/active_runtime.json ~/.config/openvr/openvrpaths.vrpath

You will want to delete or modify ~/.config/openxr/1/active_runtime.json. Note that this is an override for /etc/xdg/openxr/1/active_runtime.json, so if you are using services.monado.defaultRuntime = true (or the WiVRn equivalent) then you can safely delete this file. Otherwise, you'll want to point it to your Monado installation, which will be somewhere in /nix/store.

You will also want to modify ~/.config/openvr/openvrpaths.vrpath. Remove any runtimes that point to Envision's compiled binaries.

It is recommend to use Home Manager to automate writing these config files as mentioned above, as these paths will change regularly, due to the nature of NixOS.

## VRChat and Resonite

Your time zone may not appear correctly, to fix this you can add unset TZ to your launch arguments (i.e. env -u TZ %command%).

## Community Overlays

- nixpkgs-xr: provides overlays for the existing VR-related nixpkgs to use their git version, as well as adding a few new ones (see their readme for a full list). If the mainline packages are broken for you (whether it be compiling or in function) for any reason, adding this, as described in the readme, might help. It also provides XR-related packages considerably before they are available in NixOS.

================================================================================
PAGE 5: Performance (General)
Source: https://lvra.gitlab.io/docs/performance/
================================================================================

# Performance

## A word on antialiasing

Antialiasing should be avoided when possible and the compositor scale should be increased in Monado to smooth out edges. In general AA is found to increase the latency of delivered frames quite noticeably and the GPU utilization far more than simple super sample of the actual VR session or app.

## Set AMD GPU power profile mode

AMD GPUs will attempt to power save in between rendering frames, for flatscreen games this is helpful, but for VR this is quite negatively impactful on the VR compositor's ability to timewarp frames so the user's viewport does not stutter or cause sickness.

This is step is modestly important to avoid stutter on newer kernels, if you experience stutters under large graphical load please attempt to set this up.

### Graphical Programs

Caution: These are overclocking utilities. Overclocking your GPU is potentially dangerous and if done without care it could permanently damage your hardware. Do not use multiple overclocking tools at the same time, they will conflict with each other. If you limit yourself to setting the power profile you should be fine, but don't just crank up the sliders.

#### Linux GPU Configuration And Monitoring Tool (LACT)

This application allows you to control your AMD, Nvidia, or Intel GPU on a Linux system.

- Install it through your distro's package manager (also see installation docs)
- Enable and start the service: sudo systemctl enable --now lactd
- Select your GPU in the top-left dropdown menu, make sure to pick your dedicated GPU and not your integrated one
- Create a new profile for VR gaming
  - (Optional): You can set rules for automatically switching to the VR profile
- Go to the overclocking (OC) tab and change Performance Level to Manual
- Change Power Profile Mode to VR
- Click Apply at the bottom

#### CoreCtrl

Warning: CoreCtrl is in maintenance mode. No new features or hardware support will be added. Supports AMD GPUs up to the RX 9000 series.

- Install it through your distro's package manager
- Select your GPU on the top
- Set "Performance mode" to "Advanced"
- Set "Power profile" to "VR"

## Switching profiles manually using the terminal

Warning: Do not run these scripts while using any of the above programs.

### Enable VR profile using a script

  #!/usr/bin/env bash
  # Enable manual override
  echo "manual" | sudo tee /sys/class/drm/card0/device/power_dpm_force_performance_level
  # Translate "VR" into profile number
  vr_profile=$(cat /sys/class/drm/card0/device/pp_power_profile_mode | grep ' VR ' | awk '{ print $1; }')
  # Set profile to VR
  echo $vr_profile | sudo tee /sys/class/drm/card0/device/pp_power_profile_mode

### Disable VR profile using a script

  #!/usr/bin/env bash
  # Disable manual override
  echo "auto" | sudo tee /sys/class/drm/card0/device/power_dpm_force_performance_level
  # Set profile to DEFAULT
  echo 0 | sudo tee /sys/class/drm/card0/device/pp_power_profile_mode

## Did all of this and still suffering stuttering issues, discomfort, or sickness?

You may need to increase the overhead of your compositor timewarp. Some GPUs cannot effectively meet the demands of real-time reprojection on the default value of 4 miliseconds so a safer value of 8 can be recommended if your symptoms are accompanied by 100% GPU usage:

  U_PACING_COMP_MIN_TIME_MS=8

Insert this environment variable into your monado-service launch or into your envision profile and simply start the runtime without a rebuild. Bump the value up, as appropriate, based on the absolute power of your GPU.

Lower spec users, such as those with iGPUs, should consider radically higher values here, experiment in the double digits to arrive at a number that works well from your perspective.

================================================================================
PAGE 6: VR Gear and GPUs (Hardware)
Source: https://lvra.gitlab.io/docs/hardware/
================================================================================

# Hardware

## GPU Support

Notes:

- AMD GPU users: Make sure you are using the RADV Vulkan driver. AMDVLK is deprecated, is unable to lease displays when using SteamVR/Monado, and will cause video corruption on WiVRn.
- For Nvidia proprietary drivers older than 565, the vulkan-layers must be installed in order to not crash (AUR, Fedora).
- Wired HMDs on Intel Arc GPUs: Only DisplayPort-based HMDs work, only with i915 driver. Tested: A580, A770 with HP Reverb G2 (OK), Acer AH101 (FAIL).
- Direct Display Mode doesn't work with HDMI-based HMDs.
- Xe driver currently doesn't support Direct Display Mode at all.
- Audio over DisplayPort is known to temporarily cut out whenever new audio sources spring up on PipeWire without a fix to add ALSA headroom.

GPU Support Table:

  Nvidia >= 16XX / Nvidia Open Module: VR Acceptable, Reprojection Partial, Hybrid Supported. Requires driver 565+. Noticeable display latency by default. No fix on SteamVR.
  Nvidia <= 10XX / Nvidia Closed Source: VR Acceptable, Reprojection Partial, Hybrid Supported. Requires driver 565+. No Valve Index support.
  Nvidia / Nouveau Open Source: VR Limited, Reprojection Not Viable, Hybrid Supported. Cannot reproject. No DP audio. Not recommended.
  AMD RDNA / RADV: VR Excellent, Reprojection Robust (RDNA+), Hybrid Supported. Recommended for wired VR.
  AMD GCN / RADV: VR Excellent, Reprojection Limited, Hybrid Supported. Not recommended for wired VR.
  AMD / AMDVLK or AMDGPU PRO: Not Viable. RADV preferred in all circumstances. Do not use.
  Intel / i915: VR Functional, Reprojection Robust (Limited testing), Hybrid Supported. WiVRn tested working. Graphical glitches in some games.
  Intel / Xe: No wired HMDs, Reprojection Robust (Limited testing), Hybrid Supported. WiVRn tested working. Graphical glitches.

## XR Devices

Warning: SteamVR is not recommended for any devices, due to general unreliability.

### Vive Pro quirks
- Microphone sample rate should be set at 44.1khz (48khz raises pitch).
- Creates an HDMI Output Source after startup. Use it instead of USB Audio Source.

### Vive Pro 2 quirks
- Defaults to lower resolution (3680x1836@90hz) on Monado. Kernel patches may be required.

### Valve Index quirks
- Uses GPU DisplayPort for audio. Try changing output port in pavucontrol if no audio.
- Audio output sample rate must be 48khz.
- May enter bad state on first connection or if plugged in before boot. Try replugging power connector.

### Other HMD notes
- Vive Pro Eye: HMD functional, eye tracking with ReVision
- Pimax: Initialization WIP, distortion matrix dump in progress
- Bigscreen Beyond 2e: See Bigscreen Beyond eyetracking page
- PSVR2: Functional with minor jitter. Guide available for SteamVR branch, eye tracking, pre-PC firmware (V5.00).

### Trackers
See the trackers page for face, eye, and full-body trackers.

## Desktop hangs on start of SteamVR or Monado

Only applies to kernel versions before linux-6.19!

Symptoms: Desktop freezes on first VR start after reboot, dmesg shows "ERROR dc_stream_state is NULL". Fixed in linux-6.19. Kernel patch available for 6.18 and below.

## Applying a kernel patch (for Vive Pro 2, Bigscreen Beyond, Pimax)

### Arch
- Bigscreen Beyond: Install linux-bsb AUR package
- Manual: Download patches, modify PKGBUILD, build with MAKEFLAGS="--jobs=$(nproc)" makepkg -sir --skippgpcheck

### Fedora
- Clone kernel RPM, add patches to kernel.spec, build with fedpkg local

### NixOS
  boot.kernelPatches = [ {
    name = "type what the patch is for here";
    patch = /path/to/patch/file.patch;
  } ];

### Gentoo
- Use /etc/portage/patches/sys-kernel/gentoo-sources/ directory
- Will not work with gentoo-kernel-bin
