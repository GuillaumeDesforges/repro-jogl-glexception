This repo provides a minimal reproduction of https://discourse.nixos.org/t/force-opengl-version-on-nvidia-or-force-usage-of-mesa-opengl/37436

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
