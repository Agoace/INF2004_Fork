# Raspberry Pi Pico W development environment (Fedora)

This guide reproduces the command-line environment used by this repository on
64-bit Fedora Linux. It installs the Pico SDK, Arm cross-compiler, USB flashing
with `picotool`, and a serial monitor. VS Code is optional.

## Hardware and knowledge prerequisites

- Raspberry Pi Pico W and a data-capable Micro-USB cable
- For the Lab 1 LED exercise: LED and 330 ohm resistor
- Basic C, binary/decimal/hexadecimal, bit operations, endianness, MSB and LSB
- Fully covered shoes when working in the physical lab

## 1. Install host packages

```bash
sudo dnf install git tar curl gcc gcc-c++ make cmake python3 \
  libusb1-devel pkgconf-pkg-config
sudo usermod -aG dialout "$USER"
```

Log out and back in after adding the `dialout` group.

## 2. Install the Pico SDK and examples

The revisions below match the environment tested with this repository.

```bash
mkdir -p "$HOME/pico"

git clone --branch 2.3.0 --depth 1 \
  https://github.com/raspberrypi/pico-sdk.git "$HOME/pico/pico-sdk"
git -C "$HOME/pico/pico-sdk" submodule update --init

git clone https://github.com/raspberrypi/pico-examples.git \
  "$HOME/pico/pico-examples"
git -C "$HOME/pico/pico-examples" checkout c81c855
```

## 3. Install the Arm GNU cross-compiler

This uses Arm GNU Toolchain 14.3.Rel1 without modifying system packages.

```bash
mkdir -p "$HOME/.local/opt"
tmp_dir=$(mktemp -d)

curl -fL -o "$tmp_dir/arm-toolchain.tar.xz" \
  https://developer.arm.com/-/media/Files/downloads/gnu/14.3.rel1/binrel/arm-gnu-toolchain-14.3.rel1-x86_64-arm-none-eabi.tar.xz

tar -xJf "$tmp_dir/arm-toolchain.tar.xz" -C "$HOME/.local/opt"
rm -rf "$tmp_dir"
```

## 4. Configure the shell

Append the following to `~/.bashrc`:

```bash
# Raspberry Pi Pico C/C++ development
export PICO_SDK_PATH="$HOME/pico/pico-sdk"
export PICO_BOARD="pico_w"
export PICO_TOOLCHAIN_PATH="$HOME/.local/opt/arm-gnu-toolchain-14.3.rel1-x86_64-arm-none-eabi"
export PATH="$PICO_TOOLCHAIN_PATH/bin:$HOME/.local/bin:$PATH"
```

Load it immediately:

```bash
source "$HOME/.bashrc"
```

Verify the essential tools:

```bash
cmake --version
arm-none-eabi-gcc --version
echo "$PICO_SDK_PATH"
echo "$PICO_BOARD"
```

## 5. Install USB-enabled picotool

Pico SDK 2.3.0's bundled `picotool` release contains an RP2040 null-pointer bug.
Official upstream commit `282a3ca` fixes it, so this setup pins that revision.

```bash
git clone https://github.com/raspberrypi/picotool.git "$HOME/pico/picotool"
git -C "$HOME/pico/picotool" checkout 282a3ca

cmake -S "$HOME/pico/picotool" -B "$HOME/pico/picotool/build" \
  -DPICO_SDK_PATH="$PICO_SDK_PATH" \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX="$HOME/.local"

cmake --build "$HOME/pico/picotool/build" --parallel
cmake --install "$HOME/pico/picotool/build"
picotool version
```

`picotool version` must **not** say that USB support is unavailable.

Install its Linux device-access rules:

```bash
sudo cp "$HOME/pico/picotool/udev/60-picotool.rules" \
  /etc/udev/rules.d/60-picotool.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Unplug and reconnect the Pico after installing the rules.

## 6. Install the serial monitor

Install PySerial in an isolated user environment:

```bash
python3 -m venv "$HOME/.local/share/pico-python"
"$HOME/.local/share/pico-python/bin/python" -m pip install pyserial
ln -sf "$HOME/.local/share/pico-python/bin/pyserial-miniterm" \
  "$HOME/.local/bin/pyserial-miniterm"
```

Use the stable device name rather than assuming it will always be `ttyACM0`:

```bash
ls -l /dev/serial/by-id/
pyserial-miniterm /dev/serial/by-id/usb-Raspberry_Pi_Pico_*-if00 115200
```

Exit the monitor with `Ctrl+]`.

## 7. Build Lab 1

From the repository root:

```bash
cmake -S LAB1 -B LAB1/build -DPICO_BOARD=pico_w
cmake --build LAB1/build --parallel
```

Expected outputs include:

```text
LAB1/build/basic.uf2
LAB1/build/blinky.uf2
```

Warnings in `blinky.c` are intentional parts of the exercise.

## 8. First flash

The first flash uses the ROM bootloader:

1. Unplug the Pico W.
2. Hold **BOOTSEL** while reconnecting USB.
3. Release BOOTSEL after connection.
4. Confirm `lsusb` shows `2e8a:0003 Raspberry Pi RP2 Boot`.
5. Copy the UF2 to the automatically mounted `RPI-RP2` drive.

On Fedora:

```bash
cp LAB1/build/basic.uf2 "/run/media/$USER/RPI-RP2/"
sync
```

The Pico automatically reboots into the application.

## 9. Later flashes without BOOTSEL

Firmware linked with Pico SDK USB stdio supports forced reset and loading:

```bash
cmake --build LAB1/build --target basic --parallel && \
  picotool load -f -x LAB1/build/basic.uf2
```

For another target, replace both occurrences of `basic`, for example:

```bash
cmake --build LAB1/build --target blinky --parallel && \
  picotool load -f -x LAB1/build/blinky.uf2
```

The `-f` option asks compatible running firmware to enter BOOTSEL remotely;
`-x` starts the newly loaded firmware. If the currently installed firmware does
not expose compatible USB reset support, use the physical BOOTSEL procedure
again.

## Troubleshooting

### `picotool` cannot access the device

Check the udev rules, reconnect the Pico, then run:

```bash
picotool info -f
```

Do not run normal development commands with `sudo` once the rules are installed.

### Serial port changes from `ttyACM0` to `ttyACM1`

Close old serial monitors and use `/dev/serial/by-id/...` instead of a numbered
`ttyACM` path.

### Pico does not appear as storage

It only exposes `RPI-RP2` storage in BOOTSEL mode. Normal application mode
usually appears as USB ID `2e8a:000a` and provides a serial device instead.

### Clean rebuild

```bash
rm -rf LAB1/build
cmake -S LAB1 -B LAB1/build -DPICO_BOARD=pico_w
cmake --build LAB1/build --parallel
```

## Tested versions

| Component | Version/revision |
|---|---|
| Pico SDK | 2.3.0 |
| Arm GNU Toolchain | 14.3.Rel1 / GCC 14.3.1 |
| picotool | 2.3.0 plus official fix `282a3ca` |
| CMake | 4.4.3 tested; project minimum is 3.13 |
| Host | Fedora Linux 44 x86-64 |
