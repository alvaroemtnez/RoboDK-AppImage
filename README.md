# RoboDK AppImage
This repo provides instructions to package RoboDK from the official Ubuntu installer into an AppImage, with support for the Robot Library and double-click to open .rdk files.

## Packaging RoboDK as an Appimage
1. First, download everything we need:
  - appimagetool: https://github.com/AppImage/appimagetool/releases
  - RoboDK for Ubuntu: https://robodk.com/download
2. Create the working dirs:
```
mkdir -p RoboDK.AppDir/usr/bin
mkdir -p RoboDK.AppDir/usr/share/icons/hicolor/scalable/apps/
mkdir -p RoboDK.AppDir/usr/share/icons/hicolor/128x128/apps/
mkdir -p RoboDK.AppDir/usr/share/mime/packages/
```
3. Run the RoboDK installer, choosing the newly created `RoboDK.AppDir/usr/bin` as the installation path.
4. Copy the logos to the newly created icons folder and to `RoboDK.AppDir`:
```
cp RoboDK.AppDir/usr/bin/logo-robodk.png RoboDK.AppDir/usr/share/icons/hicolor/128x128/apps/logo-robodk.png
cp RoboDK.AppDir/usr/bin/Icons/station.svg RoboDK.AppDir/usr/share/icons/hicolor/scalable/apps/robodk-station.svg
cp RoboDK.AppDir/usr/bin/logo-robodk.png RoboDK.AppDir/logo-robodk.png
```
5. Create the appimage AppRun script, mimetypes for .rdk and .robot and Desktop file:
```
cat <<EOF > RoboDK.AppDir/robodk.desktop
[Desktop Entry]
Type=Application
Name=RoboDK
Exec=AppRun %U
Icon=logo-robodk
Terminal=false
Categories=Graphics;Engineering;
MimeType=x-scheme-handler/robodk;application/x-rdk;application/x-robot;application/x-tool;application/x-rdkp;model/step;model/iges;model/stl;model/vrml;application/x-3ds;application/x-tgif;text/csv;text/plain;text/x-gcode;application/x-cnc;text/x-abb-rapid;text/x-urscript;
EOF

cat <<EOF > RoboDK.AppDir/usr/share/mime/packages/robodk.xml
<?xml version="1.0" encoding="UTF-8"?>
<mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
  <!-- RoboDK Specific Formats -->
  <mime-type type="application/x-rdk">
    <comment>RoboDK Station</comment>
    <glob pattern="*.rdk"/>
    <icon name="robodk-station"/>
  </mime-type>
  <mime-type type="application/x-robot">
    <comment>RoboDK Robot</comment>
    <glob pattern="*.robot"/>
    <icon name="robodk-station"/>
  </mime-type>
  <mime-type type="application/x-tool">
    <comment>RoboDK Tool</comment>
    <glob pattern="*.tool"/>
    <icon name="robodk-station"/>
  </mime-type>
  <mime-type type="application/x-rdkp">
    <comment>RoboDK Plugin Package</comment>
    <glob pattern="*.rdkp"/>
    <icon name="robodk-station"/>
  </mime-type>

  <!-- G-Code and CNC -->
  <mime-type type="application/x-cnc">
    <comment>CNC Program</comment>
    <glob pattern="*.cnc"/>
    <glob pattern="*.gcode"/>
    <glob pattern="*.nc"/>
    <glob pattern="*.nci"/>
    <glob pattern="*.ngc"/>
  </mime-type>

  <!-- Vendor Scripts -->
  <mime-type type="text/x-abb-rapid">
    <comment>ABB RAPID Script</comment>
    <glob pattern="*.mod"/>
  </mime-type>
  <mime-type type="text/x-urscript">
    <comment>UR Script</comment>
    <glob pattern="*.script"/>
    <glob pattern="*.urp"/>
  </mime-type>
</mime-info>
EOF

cat <<EOF > RoboDK.AppDir/AppRun
#!/bin/sh
# Get the path where the AppImage is mounted
HERE="\$(dirname "\$(readlink -f "\${0}")")"
BIN_DIR="\$HERE/usr/bin/bin"

# Set environment variables relative to the mount point
export LD_LIBRARY_PATH="\$BIN_DIR/lib:\$LD_LIBRARY_PATH"
export QT_PLUGIN_PATH="\$BIN_DIR/plugins"
export QT_QPA_PLATFORM_PLUGIN_PATH="\$BIN_DIR/plugins"

# Move to bin so RoboDK can find its local resources
cd "\$BIN_DIR"

# Execute RoboDK and pass ALL arguments (files) to it
exec ./RoboDK "\$@"
EOF

chmod +x RoboDK.AppDir/AppRun
```
6. Build the appimage using the downloaded appimagetool:
```
chmod +x appimagetool-x86_64.AppImage
./appimagetool-x86_64.AppImage RoboDK.AppDir
```
7. `RoboDK-x86_64.AppImage` will be generated, and then it's safe to remove `RoboDK.AppDir`.


