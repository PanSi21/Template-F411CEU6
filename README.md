# Template STM32F411CEU6 — WeAct Black Pill

> Template pronto per progetti sulla scheda **WeAct STM32F411CEU6 Black Pill**, generato con STM32CubeMX e buildato con CMake + GCC ARM Embedded.

## Hardware

| Parametro | Valore |
|---|---|
| MCU | STM32F411CEUx (Cortex-M4F) |
| Flash | 512 KB |
| SRAM | 128 KB |
| HSE | 25 MHz |
| SYSCLK | 100 MHz (PLL) |

Periferiche preconfigurate:
- **PC13** → Output (LED integrato sulla Black Pill)
- **PA0** → Input con pull-up (tasto utente)
- **PA13/PA14** → SWD (debug/flash)

## Requisiti

- [GCC ARM Embedded Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads) (`arm-none-eabi-gcc` nel PATH)
- [CMake](https://cmake.org/download/) ≥ 3.22
- [Ninja](https://ninja-build.org/)
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html) (opzionale, per modificare le periferiche)

## Build

```bash
# Configurazione Debug
cmake --preset Debug

# Build Debug
cmake --build --preset Debug

# Configurazione Release
cmake --preset Release

# Build Release
cmake --build --preset Release
```

Il file `.elf` si troverà in `build/Debug/Template-F411.elf` o `build/Release/Template-F411.elf`.

## Flash

### Con st-flash (consigliato)

```bash
st-flash write build/Debug/Template-F411.elf 0x08000000
```

### Con OpenOCD

```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg -c "program build/Debug/Template-F411.elf verify reset exit"
```

### Con STM32CubeProgrammer

Aprire STM32CubeProgrammer, collegarsi via ST-Link, caricare il file `.elf` o `.hex` e programmare.

## Struttura del Progetto

```
├── CMakeLists.txt              # CMake principale
├── CMakePresets.json           # Presets Debug/Release
├── Template-F411.ioc           # File STM32CubeMX
├── STM32F411XX_FLASH.ld        # Linker script
├── startup_stm32f411xe.s       # Startup file GCC
├── cmake/
│   ├── gcc-arm-none-eabi.cmake # Toolchain GCC
│   ├── starm-clang.cmake       # Toolchain Clang (alternativo)
│   └── stm32cubemx/
│       └── CMakeLists.txt      # CMake generato da CubeMX
├── Core/
│   ├── Inc/                    # Header (main.h, gpio.h, stm32f4xx_it.h, ecc.)
│   └── Src/                    # Sorgenti (main.c, gpio.c, stm32f4xx_it.c, ecc.)
└── Drivers/
    ├── CMSIS/                  # CMSIS Core + Device headers
    └── STM32F4xx_HAL_Driver/   # Driver HAL
```

## Workflow con STM32CubeMX

Questo template è progettato per essere **rigenerato** da CubeMX ogni volta che si aggiungono o modificano periferiche:

1. Aprire `Template-F411.ioc` con STM32CubeMX
2. Configurare pin, clock e periferiche desiderate
3. Generare il codice (`Project → Generate Code` o `Ctrl+S`)
4. CubeMX aggiornerà i file in `Core/` e `Drivers/` preservando il codice scritto tra i blocchi `/* USER CODE BEGIN */` / `/* USER CODE END */`
5. Il tuo codice personalizzato va scritto **esclusivamente** nei blocchi `USER CODE` per non essere sovrascritto

## Aggiungere nuovi sorgenti

Per aggiungere nuovi file `.c` al progetto, modificare il file `cmake/stm32cubemx/CMakeLists.txt`:

```cmake
# In MX_Application_Src, aggiungere il percorso del nuovo file:
set(MX_Application_Src
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Src/main.c
    ${CMAKE_CURRENT_SOURCE_DIR}/../../Core/Src/mio_file.c   # ← nuovo file
    ...
)
```

In alternativa, aggiungere nuovi file nel `CMakeLists.txt` principale nella sezione `target_sources()`.

## Note per gli Studenti

- Il codice tra `/* USER CODE BEGIN ... */` e `/* USER CODE END ... */` **non** viene sovrascritto da CubeMX
- Tutto il codice personale va scritto in quei blocchi
- Il `main.c` preconfigurato accende il LED su PC13 — usalo come punto di partenza
- Per debug, usare `printf` con SWO o un debugger (ST-Link + GDB)

## Risorse

- [STM32F411 Reference Manual (RM0383)](https://www.st.com/resource/en/reference_manual/rm0383-stm32f411xcxe-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [STM32F411 Datasheet](https://www.st.com/resource/en/datasheet/stm32f411ce.pdf)
- [CMSIS Documentation](https://arm-software.github.io/CMSIS_5/General/html/index.html)
- [WeAct Black Pill F411CE](https://github.com/WeActStudio/WeActStudio.MCUBoard)
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)
