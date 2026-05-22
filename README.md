# Game Boy Emulator

An experimental Game Boy emulator written in C with SDL2-based rendering and input handling. The project is organized as a small CMake build with a reusable static emulator library, a runnable front-end, and a Check-based test target.

## Features

- CPU, cartridge, bus, timer, DMA, interrupt, LCD, and PPU subsystems
- SDL2 windowed rendering of the Game Boy framebuffer
- SDL2_ttf support for the UI layer
- Keyboard input mapped to the Game Boy joypad
- A separate debug window that renders a tile-viewer style view of VRAM contents
- Basic unit test target built with Check

## Repository Layout

- `CMakeLists.txt`: Top-level build definition and test registration
- `lib/`: Emulator implementation, built as the `emu` static library
- `include/`: Public headers shared across the executable, library, and tests
- `gbemu/`: Command-line front-end that launches the emulator
- `tests/`: Check-based test executable
- `cmake/`: CMake helper modules and configuration templates

## Build Requirements

### Cross-platform dependencies

- CMake
- A C compiler
- SDL2
- SDL2_ttf
- Check for the test target

### Linux and other Unix-like systems

The top-level CMake files call `find_package(SDL2 REQUIRED)` and `find_package(SDL2_ttf REQUIRED)`, so those packages must be installed in a way CMake can discover them.

The test target also uses Check, so the Check development package must be available.

### Windows

The build expects dependency folders next to the repository root:

- `../windows_deps/sdl2`
- `../windows_deps/sdl2_ttf`
- `../windows_deps/check`

The Windows build copies the SDL2 and SDL2_ttf runtime DLLs into the executable directory after build.

## Build

Configure and build with CMake from the repository root:

```bash
cmake -S . -B build
cmake --build build
```

If you want a different generator or build type, pass the usual CMake options such as `-G`, `-A`, or `-DCMAKE_BUILD_TYPE=Release`.

Note that the current top-level `CMakeLists.txt` still uses an older compatibility layout, so recent CMake versions may stop during configuration before they reach dependency detection. If that happens, the project file needs a small CMake modernization before a clean configure will succeed.

## Run

Launch the emulator by passing a ROM file path:

```bash
./build/gbemu/gbemu path/to/rom.gb
```

On Windows, the executable is built in the CMake output directory, so run the generated `gbemu.exe` from there.

At startup the emulator opens the main display window and a separate debug window. If the ROM cannot be loaded, the process exits with an error.

## Controls

Keyboard input is mapped as follows:

- `X`: A button
- `Z`: B button
- `Enter`: Start
- `Tab`: Select
- Arrow keys: D-pad

Closing the window sets the emulator shutdown flag and exits the main loop.

## Testing

Build and run the test suite with CTest:

```bash
ctest --test-dir build
```

The current test coverage is intentionally small and includes a smoke test around CPU stepping.

## Architecture Overview

The runtime is split into a small set of cooperating components:

- `emu`: orchestrates cartridge loading, UI startup, CPU execution, and the main event loop
- `cpu`: fetches and executes instructions
- `bus`: routes reads and writes to the appropriate device
- `cart`: loads ROM data and handles battery-backed save behavior
- `ppu`: produces the frame buffer and handles scanline/tile rendering logic
- `lcd`: tracks LCD state and mode flags
- `timer`: advances timer registers and triggers timer-related behavior
- `dma`: handles OAM DMA transfers
- `gamepad`: exposes joypad state to the emulated hardware
- `ui`: initializes SDL, renders the framebuffer, and handles keyboard/window events
- `dbg`: includes a simple debug text buffer hook used by the emulator internals

The front-end starts the CPU in a worker thread while the UI loop keeps processing SDL events and refreshing the windows.

## Notes

- The project currently builds as a debug-oriented emulator and hard-codes `CMAKE_BUILD_TYPE` to `Debug` in the top-level CMake file.
- The UI depends on SDL2 and SDL2_ttf at runtime.
- The repository includes a bundled font file, `NotoSansMono-Medium.ttf`, which is copied next to the executable after build.

## License

See `COPYING-CMAKE-SCRIPTS.txt` and the source files for the project license details.