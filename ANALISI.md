# Analisi Tecnica — Template STM32F411CEU6

> Analisi dettagliata del progetto `.ioc` generato da STM32CubeMX per la WeAct Black Pill STM32F411CE.

---

## 1. Configurazione Hardware

### MCU

| Parametro | Valore |
|---|---|
| Part Number | STM32F411CEU6 |
| Famiglia | STM32F4 |
| Package | UFQFPN48 |
| Core | ARM Cortex-M4F (FPU hardware) |
| Flash | 512 KB |
| SRAM | 128 KB |

### Clock Tree

| Clock | Frequenza | Sorgente |
|---|---|---|
| HSE | 25 MHz | Oscillatore esterno (sulla Black Pill) |
| HSI | 16 MHz | Oscillatore interno (non usato) |
| LSE | 32.768 kHz | Oscillatore RTC esterno |
| LSI | 32 kHz | Oscillatore interno low-speed |
| PLL M | 25 | Divisore ingresso PLL |
| PLL N | 200 | Moltiplicatore PLL |
| PLL P | 2 | Divisore uscita SYSCLK |
| VCO Input | 1 MHz | HSE / M |
| VCO Output | 200 MHz | (HSE / M) × N |
| **SYSCLK** | **100 MHz** | VCO / P |
| HCLK (AHB) | 100 MHz | SYSCLK / 1 |
| PCLK1 (APB1) | 50 MHz | HCLK / 2 |
| PCLK2 (APB2) | 100 MHz | HCLK / 1 |
| PLLQ (USB/SDIO) | 50 MHz | VCO / 4 |

**Nota:** La configurazione PLL (M=25, N=200, P=2) con HSE da 25 MHz produce correttamente 100 MHz di SYSCLK. La Black Pill WeAct ha un cristallo HSE da 25 MHz, non il più comune 8 MHz.

### Periferiche Configurate

| Pin | Funzione | Configurazione | Note |
|---|---|---|---|
| PC13 | GPIO Output | Push-pull, no pull, low speed | LED integrato sulla Black Pill (attivo LOW) |
| PA0 | GPIO Input | Pull-up | Tasto utente KEY |
| PH0 | RCC_OSC_IN | HSE | Oscillatore 25 MHz |
| PH1 | RCC_OSC_OUT | HSE | Oscillatore 25 MHz |
| PC14 | RCC_OSC32_IN | LSE | Oscillatore 32.768 kHz |
| PC15 | RCC_OSC32_OUT | LSE | Oscillatore 32.768 kHz |
| PA13 | SYS_JTMS-SWDIO | Serial Wire | Debug SWD |
| PA14 | SYS_JTCK-SWCLK | Serial Wire | Debug SWD |

### NVIC

- **Priority Group:** 4 (4 bit preemption, 0 sub-priority → 16 livelli di priorità)
- **Interrupt abilitati:** HardFault, MemManage, BusFault, UsageFault, NMI, SVCall, PendSV, SysTick

---

## 2. Struttura del Progetto

```
Template_F411CEU6_BLACKPILL/
├── CMakeLists.txt                  # CMake principale (non rigenerato)
├── CMakePresets.json               # Presets: Debug/Release
├── Template-F411.ioc               # File STM32CubeMX (configurazione)
├── STM32F411XX_FLASH.ld            # Linker script
├── startup_stm32f411xe.s           # Startup assembly GCC
├── .mxproject                      # Metadati interni CubeMX
├── cmake/
│   ├── gcc-arm-none-eabi.cmake     # Toolchain GCC ARM
│   ├── starm-clang.cmake           # Toolchain ST ARM Clang (alternativo)
│   └── stm32cubemx/
│       └── CMakeLists.txt          # CMake autogenerato da CubeMX
├── Core/
│   ├── Inc/
│   │   ├── main.h                  # Definizioni e pin macro
│   │   ├── gpio.h                  # Prototipi GPIO
│   │   ├── stm32f4xx_it.h          # Prototipi interrupt handler
│   │   └── stm32f4xx_hal_conf.h    # Configurazione moduli HAL
│   └── Src/
│       ├── main.c                  # Entry point + clock config
│       ├── gpio.c                  # Inizializzazione GPIO
│       ├── stm32f4xx_it.c          # Interrupt handlers
│       ├── stm32f4xx_hal_msp.c     # MCU Specific Package (HAL)
│       ├── system_stm32f4xx.c      # SystemInit() + clock table
│       ├── syscalls.c              # Syscalls C library (printf, ecc.)
│       └── sysmem.c                # Heap management (sbrk)
└── Drivers/
    ├── CMSIS/
    │   ├── Include/                # CMSIS Core headers (tutti i core ARM)
    │   └── Device/ST/STM32F4xx/    # Header specifici STM32F4
    └── STM32F4xx_HAL_Driver/
        ├── Inc/                    # Header HAL + LL
        └── Src/                    # Sorgenti HAL (solo moduli usati)
```

---

## 3. Analisi Driver HAL

### Moduli Abilitati (`stm32f4xx_hal_conf.h`)

| Modulo | Stato | File .c incluso |
|---|---|---|
| HAL_GPIO_MODULE | ✅ | `stm32f4xx_hal_gpio.c` (19 KB) |
| HAL_EXTI_MODULE | ✅ | `stm32f4xx_hal_exti.c` (15 KB) |
| HAL_DMA_MODULE | ✅ | `stm32f4xx_hal_dma.c` + `*_ex.c` (50 KB) |
| HAL_RCC_MODULE | ✅ | `stm32f4xx_hal_rcc.c` + `*_ex.c` (195 KB) |
| HAL_FLASH_MODULE | ✅ | `stm32f4xx_hal_flash.c` + `*_ex.c` + `*_ramfunc.c` (79 KB) |
| HAL_PWR_MODULE | ✅ | `stm32f4xx_hal_pwr.c` + `*_ex.c` (45 KB) |
| HAL_CORTEX_MODULE | ✅ | `stm32f4xx_hal_cortex.c` (19 KB) |
| HAL_MODULE (core) | ✅ | `stm32f4xx_hal.c` (19 KB) |

**Totale sorgenti HAL compilati:** ~441 KB

### Moduli Presenti ma DISABILITATI

I seguenti moduli sono elencati in `stm32f4xx_hal_conf.h` ma commentati (non compilati):
ADC, CAN, CRC, DAC, I2C, I2S, SPI, TIM, UART, USART, RTC, RNG, SD, USB OTG, WWDG, IWDG, e molti altri.

**Nota importante:** I file `.h` sono comunque presenti su disco e accessibili via include path. Quando si abiliterà un nuovo modulo da CubeMX, il `.c` corrispondente verrà automaticamente aggiunto al CMakeLists.txt.

### Header LL (Low-Layer)

Sono presenti anche i driver LL (`stm32f4xx_ll_*.h`), che sono wrapper diretti sui registri con overhead minimo. Non vengono compilati come `.c` (sono header-only), ma possono essere inclusi nel codice quando serve accesso diretto ai registri senza l'overhead HAL.

### Directory Legacy

`Drivers/STM32F4xx_HAL_Driver/Inc/Legacy/stm32_hal_legacy.h` è presente e inclusa nell'include path. Contiene macro deprecate per compatibilità con versioni precedenti delle HAL. Non viene usato attivamente.

---

## 4. Analisi CMSIS

### File Necessari per STM32F411 (Cortex-M4F)

| File | Dimensione | Scopo |
|---|---|---|
| `core_cm4.h` | 119 KB | Core header per Cortex-M4 (include FPU) |
| `cmsis_compiler.h` | 9.3 KB | Astrazione compiler |
| `cmsis_gcc.h` | 62 KB | Definizioni specifiche GCC |
| `cmsis_version.h` | 1.7 KB | Versione CMSIS |
| `mpu_armv7.h` | 12 KB | MPU per ARMv7-M |
| `stm32f411xe.h` | 633 KB | **Tutti i registri del dispositivo** |
| `stm32f4xx.h` | 12 KB | Header principale STM32F4 |
| `system_stm32f4xx.h` | 2.2 KB | Dichiarazione SystemInit |

### File NON Necessari (altri core ARM)

I seguenti file sono presenti ma **mai inclusi** per il Cortex-M4. Occupano circa **1.6 MB** (~89% della cartella CMSIS/Include):

| File | Core di destinazione |
|---|---|
| `core_cm0.h` | Cortex-M0 |
| `core_cm0plus.h` | Cortex-M0+ |
| `core_cm1.h` | Cortex-M1 |
| `core_cm3.h` | Cortex-M3 |
| `core_cm7.h` | Cortex-M7 |
| `core_cm23.h` | Cortex-M23 |
| `core_cm33.h` | Cortex-M33 |
| `core_cm35p.h` | Cortex-M35P |
| `core_cm55.h` | Cortex-M55 |
| `core_cm85.h` | Cortex-M85 |
| `core_sc000.h` | SC000 |
| `core_sc300.h` | SC300 |
| `core_armv8mbl.h` | ARMv8-M Baseline |
| `core_armv8mml.h` | ARMv8-M Mainline |
| `core_armv81mml.h` | ARMv8.1-M Mainline |
| `core_starmc1.h` | STAR-MC1 |
| `mpu_armv8.h` | ARMv8-M MPU |
| `cachel1_armv7.h` | L1 Cache |
| `pac_armv81.h` | Pointer Auth |
| `pmu_armv8.h` | PMU |
| `tz_context.h` | TrustZone |

**Perché non eliminarli?** STM32CubeMX rigenera l'intero pacchetto CMSIS ad ogni generazione. Eliminarli manualmente richiederebbe di ripulirli ogni volta. Inoltre, l'impatto sul build è nullo (non vengono compilati) e sul repo Git sono compressi. Per un repo pubblico didattico, averli tutti è più pratico.

---

## 5. Linker Script (`STM32F411XX_FLASH.ld`)

### Mappa di Memoria

| Regione | Indirizzo Inizio | Dimensione |
|---|---|---|
| FLASH (rx) | 0x08000000 | 512 KB |
| RAM (xrw) | 0x20000000 | 128 KB |

### Stack e Heap

| Sezione | Dimensione | Valore |
|---|---|---|
| Stack | 0x400 | 1024 byte (1 KB) |
| Heap | 0x200 | 512 byte |

**Criticità:**
- **Heap da 512 byte è molto piccolo.** Se si usa `malloc()`, `new` (C++), o funzioni che allocano dinamicamente, si esaurisce rapidamente. Per uso didattico base va bene, ma per progetti più complessi si consiglia di portarlo a 0x400 (1 KB) o più.
- **Stack da 1 KB** è adeguato per programmi semplici, ma può essere insufficiente con interrupt annidati o RTOS.
- Entrambi i valori sono configurabili in CubeMX: `Project Manager → Project → Minimum Stack/Heap Size`.

### Sezioni

- `.isr_vector` — Tabella interrupt vector (all'inizio della FLASH)
- `.text` — Codice eseguibile
- `.rodata` — Dati read-only (const, stringhe)
- `.data` — Dati inizializzati (caricati in FLASH, copiati in RAM all'avvio)
- `.bss` — Dati non inizializzati (azzerati all'avvio)
- `.tdata` / `.tbss` — Thread-Local Storage (supporto TLS completo)
- `._user_heap_stack` — Heap e stack dell'utente

**Nota TLS:** Il linker script include un supporto completo per Thread-Local Storage (TLS), utile se si usa un RTOS o C++ con `thread_local`.

---

## 6. Toolchain

### GCC ARM Embedded (default)

File: `cmake/gcc-arm-none-eabi.cmake`

| Impostazione | Valore |
|---|---|
| CPU | `-mcpu=cortex-m4` |
| FPU | `-mfpu=fpv4-sp-d16` |
| ABI | `-mfloat-abi=hard` (FPU hardware) |
| C Standard | C11 (con estensioni GCC) |
| Debug | `-O0 -g3` |
| Release | `-Os -g0` |
| C Library | `--specs=nano.specs` (newlib-nano, ridotta) |
| Linker flags | `--gc-sections` (rimozione sezioni inutilizzate) |
| Stack usage | `-fstack-usage` (genera file `.su`) |

### ST ARM Clang (alternativo)

File: `cmake/starm-clang.cmake`

Supporta tre modalità:
- **STARM_HYBRID** (default): compilatore starm-clang + linker GNU con nano.specs
- **STARM_NEWLIB**: starm-clang con newlib completo
- **STARM_PICOLIBC**: starm-clang con picolibc (più leggera di newlib-nano)

I file `syscalls.c` e `sysmem.c` includono già il supporto per picolibc con `__strong_reference` aliases.

---

## 7. File di Sistema

### `syscalls.c` (5.0 KB)

Implementa le syscall della C library per redirect di I/O:
- `_read`, `_write` → redirect su UART/semihosting
- `_sbrk` → gestione heap (delegata a `sysmem.c`)
- `_close`, `_lseek`, `_fstat`, `_isatty`, `_getpid`, `_kill`, `_exit`, ecc.

**Stato attuale:** Le funzioni `_write` e `_read` sono stub vuoti. Per usare `printf`, bisogna implementare `_write` per instradare l'output su UART.

### `sysmem.c` (3.0 KB)

Implementa `_sbrk` per la gestione dell'heap. Usa le variabili `end` e `_end` dal linker script come punto di partenza dell'heap.

### `system_stm32f4xx.c` (27 KB)

Contiene `SystemInit()`, chiamata dall'assembly di startup prima del `main()`:
- Configura il coprocessore FPU (se presente)
- Configura il Vector Table Offset (VTOR)
- **NON** configura i clock → il clock viene configurato da `SystemClock_Config()` nel `main.c` tramite le HAL
- Include una tabella commentata con le configurazioni clock per diverse combinazioni HSE/HSI/PLL

### `startup_stm32f411xe.s` (20 KB)

Assembly di startup GCC:
- Inizializza lo stack pointer
- Chiama `SystemInit()`
- Copia `.data` da FLASH a RAM
- Azzera `.bss`
- Chiama `__libc_init_array` (costruttori C++)
- Salta a `main()`
- Definisce la tabella dei vettori di interrupt completa per STM32F411
- Tutti gli handler sono weak alias a `Default_Handler` (loop infinito)

---

## 8. CMake

### `CMakePresets.json`

Due preset disponibili:

| Preset | Build Type | Ottimizzazione |
|---|---|---|
| Debug | Debug | `-O0 -g3` |
| Release | Release | `-Os -g0` |

Binari in `build/Debug/` e `build/Release/`.

### `cmake/stm32cubemx/CMakeLists.txt`

Generato automaticamente da CubeMX. Contiene:
- `MX_Defines_Syms` → macro di compilazione (`USE_HAL_DRIVER`, `STM32F411xE`)
- `MX_Include_Dirs` → include paths
- `MX_Application_Src` → sorgenti applicazione (main.c, gpio.c, ecc.)
- `STM32_Drivers_Src` → sorgenti HAL driver
- Crea libreria `STM32_Drivers` (OBJECT) e target `stm32cubemx` (INTERFACE)

**Attenzione:** Questo file viene sovrascritto da CubeMX. Non modificarlo manualmente se vuoi rigenerare il progetto. Per aggiungere sorgenti custom, modifica il `CMakeLists.txt` principale.

---

## 9. Miglioramenti Suggeriti

### Per l'IOC (CubeMX)

| Voce | Attuale | Consigliato | Motivazione |
|---|---|---|---|
| Heap Size | 0x200 (512 B) | 0x400 (1 KB) minimo | `malloc` e allocazioni dinamiche necessitano più spazio |
| USE_FULL_ASSERT | Disabilitato | Abilitato in Debug | Utile per studenti: cattura errori di configurazione HAL |
| LSE | Abilitato | Disabilitare se non si usa RTC | Risparmio energia e semplificazione clock tree |
| Compiler Optimize | 6 (Size) | 3 (Balanced) per Debug | Ottimizzazione "size" può rendere il debug più difficile |

### Per il Progetto

| Voce | Note |
|---|---|
| `printf` non funzionante | Implementare `_write` in `syscalls.c` per instradare output su UART |
| Nessun esempio di blink | Il `main.c` generato ha loop vuoto — aggiungere toggle LED in USER CODE |
| HAL_RCC_Ex molto grande | `stm32f4xx_hal_rcc_ex.c` pesa 153 KB — se non servono clock avanzati, si può ottimizzare |
| Ethernet config in hal_conf | Il blocco Ethernet (righe 204-258) è boilerplate inutile per F411 (non ha periferica Ethernet) |

### Per il Repo GitHub

| Voce | Note |
|---|---|
| `.gitignore` | ✅ Creato — esclude build artifacts e file interni |
| `.mxproject` | ❌ Da escludere (file interno CubeMX, già nel .gitignore) |
| `.ioc` | ✅ Da mantenere — serve per rigenerare il progetto |
| LICENSE | Da aggiungere (MIT, BSD, o Apache per progetti didattici) |

---

## 10. Roadmap: Transizione verso CMSIS Puro (Corso Bare Metal)

Quando si vorrà passare da HAL a CMSIS puro, ecco il percorso graduale:

### Fase 1: Ridurre HAL al minimo
1. Da CubeMX, disabilitare tutti i moduli HAL non strettamente necessari
2. In `stm32f4xx_hal_conf.h`, commentare tutti i `HAL_*_MODULE_ENABLED` tranne quelli essenziali
3. Rimuovere i `.c` HAL non necessari dal `cmake/stm32cubemx/CMakeLists.txt`

### Fase 2: Sostituire inizializzazioni HAL con CMSIS
1. Riscrivere `SystemClock_Config()` usando registri RCC diretti:
   ```c
   RCC->CR |= RCC_CR_HSEON;        // Attiva HSE
   while (!(RCC->CR & RCC_CR_HSERDY)); // Attendi ready
   RCC->PLLCFGR = ...;              // Configura PLL
   RCC->CR |= RCC_CR_PLLON;         // Attiva PLL
   // ...
   ```
2. Riscrivere `MX_GPIO_Init()` usando registri GPIO:
   ```c
   RCC->AHB1ENR |= RCC_AHB1ENR_GPIOCEN;
   GPIOC->MODER |= GPIO_MODER_MODE13_0;
   GPIOC->ODR ^= GPIO_ODR_OD13;
   ```
3. Sostituire `HAL_Delay()` con SysTick diretto o loop calibrated

### Fase 3: Rimuovere HAL completamente
1. Rimuovere `#include "stm32f4xx_hal.h"` da `main.h` → usare solo `stm32f411xe.h`
2. Rimuovere `HAL_Init()` dal `main()`
3. Riscrivere `SystemInit()` per configurare clock direttamente
4. Rimuovere tutti i driver HAL dal CMakeLists
5. Rimuovere `stm32f4xx_hal_conf.h`
6. Tenere solo: CMSIS Core, Device header, startup, linker script, syscalls

### File CMSIS già pronti (non servono modifiche)
- `Drivers/CMSIS/Include/core_cm4.h` — Core Cortex-M4
- `Drivers/CMSIS/Device/ST/STM32F4xx/Include/stm32f411xe.h` — Tutti i registri
- `startup_stm32f411xe.s` — Startup (già CMSIS-compatible)
- `STM32F411XX_FLASH.ld` — Linker script (indipendente da HAL)

---

## 11. Dimensioni Stimate del Firmware

Basandosi sui file HAL attualmente compilati:

| Componente | Dimensione stimata |
|---|---|
| Startup + SystemInit | ~2 KB |
| HAL RCC + RCC_Ex | ~12 KB |
| HAL GPIO | ~4 KB |
| HAL FLASH + Ex + Ramfunc | ~8 KB |
| HAL PWR + Ex | ~5 KB |
| HAL DMA + Ex | ~8 KB |
| HAL CORTEX | ~4 KB |
| HAL EXTI | ~3 KB |
| HAL Core | ~3 KB |
| C Library (newlib-nano) | ~8 KB |
| **Totale stimato** | **~57 KB** (~11% della FLASH) |

Con CMSIS puro, la dimensione scenderebbe a ~2-4 KB per un blink LED base.
