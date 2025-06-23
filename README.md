# STM32F401RE SWV Debug Setup (No .ioc, Bare-metal)

This is a minimal STM32CubeIDE project demonstrating how to use **SWV (Serial Wire Viewer)** on the STM32F401RE **without any `.ioc` file**. It uses `ITM_SendChar()` to redirect `printf()` for real-time debugging over SWO.

---

## ✅ Features

- SWV printf debugging using ITM Stimulus Port 0
- No `.ioc` or CubeMX required — manually configured project
- Verified on **STM32F401RE** (NUCLEO board)
- Suitable for performance profiling or bare-metal logging

---

## 🔧 Project Structure

```
SWV_LEARNING/
├── Core/
│   ├── Inc/
│   |
│   └── Src/
│       ├── main.c              # Main program loop
│       └── syscalls.c          # Contains SWV_Init, ITM_SendChar, _write override
├── .gitignore
├── README.md
└── STM32CubeIDE auto files
```

---

## 🧩 Function Placement & Integration Guide

### 📂 File: `Src/syscalls.c`

Paste the following macros at the top of the file:

```c
#define DEMCR         (*(volatile uint32_t*)0xE000EDFC)
#define ITM_TCR       (*(volatile uint32_t*)0xE0000E80)
#define ITM_TER       (*(volatile uint32_t*)0xE0000E00)
#define ITM_PORT0_U32 (*(volatile uint32_t*)0xE0000000)
```

Paste this below your includes:

```c
void SWV_Init(void)
{
    // Enable trace and ITM
    DEMCR |= (1 << 24); // TRCENA

    ITM_TCR = (1 << 0)  |  // ITMENA
              (1 << 1)  |  // TSENA
              (1 << 2)  |  // SYNCENA
              (1 << 3)  |  // DWTENA
              (1 << 4);    // SWOENA

    ITM_TER |= (1 << 0);   // Enable stimulus port 0
}

void ITM_SendChar(uint8_t ch)
{
    if (!(ITM_TCR & 1) || !(ITM_TER & 1)) return;  // Not enabled
    while (!(ITM_PORT0_U32 & 1));                  // Wait until ready
    ITM_PORT0_U32 = ch;
}

int _write(int file, char *ptr, int len)
{
    for (int i = 0; i < len; i++) {
        ITM_SendChar(*ptr++);
    }
    return len;
}
```

---

### 📂 File: `Src/main.c`

In your `main()` function, call `SWV_Init();` once:

```c
int main(void)
{
    
    SWV_Init(); // ← Required for SWV to work!

    printf("Hello from STM32 via SWV!\n");

    while (1)
    {
        printf("Tick \n");
    
    }
}
```

✅ After this, `printf()` will output to **SWV ITM Console** in STM32CubeIDE.

---

## 🛠️ How to Use in STM32CubeIDE

1. Open the project in **STM32CubeIDE**
2. Go to:
   ```
   Run > Debug Configurations > Your Project > Debugger Tab
   ```
3. Enable:
   - ✅ **Serial Wire Viewer**
   - Set **Core Clock** to `84 MHz`
   - Set **SWO Clock Prescaler** to `Auto` or `2000 kHz`
4. Start Debugging
5. Open **SWV ITM Console** via:
   ```
   Window > Show View > SWV > SWV ITM Console
   ```
6. Click ➕ Add Port 0 → Click ▶ Start Trace

---

## 🧪 Expected Output

```
Hello from STM32 via SWV!
Tick
Tick
...
```

---

## 🧰 Tools Used

- STM32CubeIDE v1.17.0
- STM32F401RE NUCLEO board
- ST-Link debugger
- ARM Cortex-M4 DWT, ITM, DEMCR

---

## 📝 License

MIT License — feel free to use, fork, and adapt.

> Maintained by [@muhammedsaifudeen](https://github.com/muhammedsaifudeen)
}
