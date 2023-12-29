This repo provides a minimal reproduction on NixOS of the error

```
Exception in thread "main" com.jogamp.opengl.GLException: Profile GL2 is not available on null, but: []
	at com.jogamp.opengl.GLProfile.get(GLProfile.java:1009)
	at com.jogamp.opengl.GLProfile.get(GLProfile.java:1022)
	at fr.guilllaumedesforges.jogl.helloworld.App.main(App.java:12)
```

given

```
$ glxinfo -B
name of display: :0
display: :0  screen: 0
direct rendering: Yes
Memory info (GL_NVX_gpu_memory_info):
    Dedicated video memory: 6144 MB
    Total available memory: 6144 MB
    Currently available dedicated video memory: 5009 MB
OpenGL vendor string: NVIDIA Corporation
OpenGL renderer string: NVIDIA GeForce GTX 1660 SUPER/PCIe/SSE2
OpenGL core profile version string: 4.6.0 NVIDIA 545.29.02
OpenGL core profile shading language version string: 4.60 NVIDIA
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile

OpenGL version string: 4.6.0 NVIDIA 545.29.02
OpenGL shading language version string: 4.60 NVIDIA
OpenGL context flags: (none)
OpenGL profile mask: (none)

OpenGL ES profile version string: OpenGL ES 3.2 NVIDIA 545.29.02
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
```

See https://discourse.nixos.org/t/force-opengl-version-on-nvidia-or-force-usage-of-mesa-opengl/37436

```console
# install tools
nix shell nixpkgs#openjdk nixpkgs#maven

# build
mvn package

# libs shenanigans
jar xvf target/jogl-helloworld-1.0-SNAPSHOT.jar natives
nix build --no-link --impure --expr 'let pkgs = (__getFlake "nixpkgs").legacyPackages.${builtins.currentSystem};in with pkgs; [ libGL xorg.libX11 xorg.libXxf86vm xorg.libXrender ]'
export LD_LIBRARY_PATH="$(nix eval --impure --expr 'let pkgs = (__getFlake "nixpkgs").legacyPackages.${builtins.currentSystem};in pkgs.lib.strings.makeLibraryPath (with pkgs; [ libGL xorg.libX11 xorg.libXxf86vm xorg.libXrender ])'):$(realpath natives/linux-amd64)"
# ^--- replace linux-amd64 by your platform

# run
java -Djogamp.debug=1 -jar target/jogl-helloworld-1.0-SNAPSHOT.jar
```
