# Aux

Aux is a local voice assistant built around an Arduino UNO Q. It listens for a
voice request, reasons with a small local language model, controls real-world
devices, speaks a response, and shows its state on the onboard LED matrix.

## Architecture

```text
Microphone
  -> speech-to-text (Python/Linux)
  -> local small LLM (Python/Linux)
  -> action router
       -> Govee LAN API
       -> Bridge RPC -> Arduino MCU firmware -> LEDs / matrix / GPIO
  -> text-to-speech (Python/Linux)
  -> Bluetooth speaker

TTS audio level -> Bridge RPC -> LED-matrix voice visualizer
```

The UNO Q's Debian/Linux processor runs the high-level application. Its
STM32/Zephyr microcontroller handles immediate hardware work, matrix rendering,
and other timing-sensitive I/O.

## Project layout

```text
jarvis-core/       Python voice, AI, TTS, and smart-home orchestration
jarvis-firmware/   Arduino C++ firmware for the UNO Q MCU
docs/              Setup notes, supported devices, and wiring details
```

## Plan

1. Set up the UNO Q Linux environment, Wi-Fi, Bluetooth speaker, and a USB
   microphone.
2. Create `jarvis-firmware` with a Bridge RPC interface and a basic LED-matrix
   visualizer that accepts a voice level from 0 to 7.
3. Create `jarvis-core` with a small test program that calls the MCU through
   Bridge and animates the matrix.
4. Add text-to-speech and feed the generated audio's RMS level to the matrix
   while it plays.
5. Add speech-to-text with push-to-talk as the initial interaction mode.
6. Add a small quantized local LLM and a constrained action/intent layer.
7. Add Govee control using the local LAN API; retain cloud control only as a
   fallback for unsupported devices.
8. Add assistant states: idle, listening, thinking, speaking, success, and
   error.
9. Harden the system: command confirmation for destructive actions, error
   recovery, logging, and auto-start on boot.

## First milestone

Press a button or run a Python command, have the MCU display a speaking-style
visualizer, and play a test phrase through the Bluetooth speaker. This proves
the Python -> Bridge -> MCU -> matrix and Linux -> Bluetooth audio paths before
adding AI.

## Design principles

- Keep the MCU firmware small, deterministic, and hardware-focused.
- Keep audio, AI, networking, and Govee integration on Linux in Python.
- Prefer local network control for lights whenever the device supports it.
- Start with explicit, safe actions before allowing open-ended LLM tool use.
