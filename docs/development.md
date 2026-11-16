* Windows

Update the vcpkg:

``` pwsh
vcpkg x-update-baseline
```

Build on Windows:

``` pwsh
$env:VCPKG_ROOT = Split-Path (Get-Command vcpkg).Source
cmake -GNinja -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo `
  "-DCMAKE_TOOLCHAIN_FILE=$env:VCPKG_ROOT\scripts\buildsystems\vcpkg.cmake" `
  -DBUILD_SAMPLES=OFF  ../
```