This example is extended from [CMSIS-Zephyr/Alif_E7_Zephyr/ipm_arm_mhuv2](https://github.com/Arm-Examples/CMSIS-Zephyr/tree/main/Alif_E7_Zephyr/ipm_arm_mhuv2) with Zephyr [System build](https://docs.zephyrproject.org/latest/build/sysbuild/index.html#sysbuild-system-build). Sysbuild is a system-level build mechanism that allows multiple cores or images to be built together in one unified process. It manages configuration and artifacts for all domains (i.e., projects), so developers can run a single `west build --sysbuild` instead of handling each core separately.

To enable SysBuild with multiple domains, a `sysbuild.cmake` file in the primary domain (in this case, **rtss_he**) has to be provided. This file registers the additional project **rtss_hp** so it can be built together:
```
ExternalZephyrProject_Add(
    APPLICATION rtss_hp
    SOURCE_DIR ../rtss_hp
    BOARD alif_e7_dk_rtss_hp
)
```
- **APPLICATION** specifies the secondary domain (i.e., the project name).
- **SOURCE_DIR** points to the source folder of the additional domain.
- **BOARD** selects the target board for that domain (optional if the same as the primary).

The build is then invoked from the csolution.yml configuration using the following command:
```
west build -d ../build_DualCores/ -b alif_e7_dk_rtss_he
    -p auto --sysbuild ../rtss_he --
    -Drtss_he_CONFIG_DEBUG=y -Drtss_he_CONFIG_DEBUG_THREAD_INFO=y
    -Drtss_hp_CONFIG_DEBUG=y -Drtss_hp_CONFIG_DEBUG_THREAD_INFO=y
    -Drtss_he_SE_SERVICES:BOOL=OFF -Drtss_he_RTSS_HP_MHU0:BOOL=ON
    -Drtss_hp_SE_SERVICES:BOOL=OFF -Drtss_hp_RTSS_HE_MHU0:BOOL=ON
```

The changes is that SysBuild supports [**CMake variable namespacing**](https://docs.zephyrproject.org/latest/build/sysbuild/index.html#cmake-variable-namespacing) to pass values into each domain. **Kconfig** options are expressed as `-D<domainName>_CONFIG_<var>=<value>`. Plain CMake variables use the format `-D<domainName>_<var>=<value>`. The optional type suffix such as `:BOOL` may be included or omitted.

However, there is a known issue with passing CMake variables in this way, as described in [Zephyr Issue #76592](https://github.com/zephyrproject-rtos/zephyr/issues/76592). Variables declared with the `-D<domainName>_<var>=...` form are not automatically passed to each project. The recommended workaround is to modify the application’s CMakeLists.txt and explicitly retrieve the values using `zephyr_get()`. For example:
```
zephyr_get(SE_SERVICES)
zephyr_get(RTSS_HP_MHU0)
```
After applying this workaround, both build artifacts for rtss_he and rtss_hp will be generated under the folder *./build_DualCores/* by using one `west build` command.