# Media Engine Reload: What's New from 2025

> **Important Note:** The information in this repository may differ from the latest findings. Please also refer to the [`psp-media-engine-custom-core`](https://github.com/mcidclan/psp-media-engine-custom-core) and [`psp-media-engine-cracking-the-unknown`](https://github.com/mcidclan/psp-media-engine-cracking-the-unknown) projects for additional information.

## Code Execution Setup

Code execution on the Media Engine can be achieved through the reset vector in kernel mode or using one of the new methods available in the `me-safe-task` library project. This requires kernel-level privileges through one of the following approaches:
1. **External kernel module**: Code deployed within a kernel module loaded from userland
2. **Kernel bridge module**: Small kernel module containing a bridge to handle callbacks
3. **kuBridge**: Available via custom firmware
4. **Kexploit**: Eliminates external PRX dependency but increases boot time

While using reset vector, the initialization code must be copied from System Control to it, which is located at address `0xBFC00000`. The Status Register requires proper configuration to clear exceptions and enable the FPU before running your code.

## Memory Map

### 0x00000000: EDRAM
- **PSP-1000**: 2MB internal EDRAM  
- **PSP-2000+**: 4MB internal EDRAM
- **Usage**:
 - Can be used for Color Space Conversion (CSC) processing via CSC controller
 - Source/destination for DMACplus transfers
 - Compatible with internal DMAC for DSP, and accessibe from the VME DSP
 - General-purpose local memory for custom processing
 - Local Stack

### 0x04000000: Ping-Pong Buffers
Activated by setting bit 5 (DMA A bus) of hardware register `0xBC100050` (Media Engine side).

- **Base Buffers**: Range: `0x04000000` to `0x0401ffff`  
Contains main 4 × 8KB 24-bit buffers (32-bit word aligned) + mirrors  

- **Top buffers**: Range: `0x04020000` to `0x04031fff`  
Contains main 4 × 8KB 24-bit buffers + mirrors  

### 0x040FF000: Primary DMAC
Handles data transfers between ping-pong buffers and main memory from Media Engine. Shares hardware control registers with Multiplexer DMAC but uses different configuration parameters. Additional sources/destinations require further testing.

#### Hardware registers

*Note: the following channel/address generator descriptions may be incomplete or inaccurate, as their behavior depends on the current control value. Please refer to the "psp-media-engine-cracking-the-unknown" and/or "the-psp-virtual-mobile-engine" projects for more up-to-date information.*

| Address    |  R/W | Description |
|------------|------|-------------|
| 0x440ff000 |  RW  | Status |
| 0x440ff004 |  RW  | Config / state / routing ? (values met 0x0, 0x31B, 0x831B, 0x1031B 0x1831B) |
| 0x440ff008 |  RW  | Control |
| 0x440ff00c |  --  | Unknown |
| 0x440ff010 |  RW  | Channel1 external src or dst depending on the control value |
| 0x440ff014 |  RW  | Channel1 word (4 bytes) count - 1 to be transfered |
| 0x440ff018 |  RW  | Channel1 internal src or dst word offset depending on the control value | 
| 0x440ff01c |  -W  | Clear mode ? |
| 0x440ff020 |  RW  | Channel2 src or dst word offset / addr ? | 
| 0x440ff024 |  RW  | Channel2 word (4 bytes) count - 1 to be transfered |
| 0x440ff028 |  RW  | Channel2 internal src or dst word offset depending on the control value |
| 0x440ff02c |  RW  | Constant fill value / config / offset  ? |
| 0x440ff030 |  RW  | Channel3 src or dst word offset / addr ? |
| 0x440ff034 |  RW  | Channel3 word (4 bytes) count - 1 to be transfered |
| 0x440ff038 |  RW  | Channel3 internal src or dst word offset depending on the control value |
| 0x440ff03c |  --  | Unknown |
| 0x440ff040 |  -W  | Unknown |

#### 0x440ff004 register flags
bit[4]  - necessary while using primary dmac wait functions

#### Control register flags
bit[0]  -
bit[1]  -
bit[2]  -
bit[3]  -
bit[4]  -
bit[5]  -
bit[6]  -
bit[7]  -
bit[8]  -
bit[9]  -

### 0x04100000-0x04200000: VLD Unit (Variable Length Decoder)
Activated by setting bit 6 (DMA B bus) of hardware register `0xBC100050` (Media Engine side). Hardware entropy decoder for video compression, decodes variable-length bitstreams into signed 32-bit DCT coefficients. Uses configurable code length lookup tables indexed by DCT levels to extract 1-32 bit codes from the bitstream.

#### Hardware registers
| Address    |  R/W | Description |
|------------|------|-------------|
| 0x44100000 |  RW  | Config / control / status |
| 0x44100004 |  RW  | Max processed DCT coefficient count |
| 0x44100008 |  RW  | Block step ?               3 bits |
| 0x4410000c |  RW  | DCT coefficient step       3 bits |
| 0x44100010 |  RW  | DCT level format           6 bits |
| 0x44100014 |  RW  | DCT level buffer stride exponent (offset = 1 << N) | 5 bits |
| 0x44100018 |  RW  | Unknown                    3 bits | 
| 0x4410001c |  RW  | Bitstream buffer address |
| 0x44100020 |  RW  | Bitstream cursor (DCT code position) | 
| 0x44100024 |  RW  | Config buffer ? First bytes are DCT level/LUT index and code length - 1 depending on config |
| 0x44100028 |  RW  | Code length LUT (DCT level indexed) |
| 0x4410002c |  RW  | Max DCT code count ? |
| 0x44100030 |  RW  | DCT coefficient output buffer address |
| 0x44100034 |  RW  | Unknown |
| 0x44100038 |  RW  | Unknown |
| 0x4410003c |  RW  | Transfer mode ? |

#### Config register flags
bit[0]  - Start decode  
bit[6]  - Process DCT level ?  
bit[7]  - ?  
bit[8]  - ?  
bit[9]  - ?  
bit[10] - Enable cursor  
bit[16] - ?  

Examples:
0b00000010000000000000001011000001  // 0x020002C1  
0b00000000000000010000010111000001  // 0x000105C1  
0b00000000000000010000011011000001  // 0x000106C1  

### 0x040F8000: VME Reconfigurable DataPath

It is now confirmed that it can handle at least 8 sources and 4 destinations across 24-bit buffers, with possible routing to other inputs (likely audio FIFO outputs). Activated with DMA A bit. The 4 main hardware config registers are PE operation descriptors.

Please see [Cracking The Unknown Project](https://github.com/mcidclan/psp-media-engine-cracking-the-unknown) for more information on the VME DSP

## Components

### DMACplus
DMACplus provides bidirectional DMA transfers between Media Engine and Main CPU, managed by the main system control.

### DMACplusLCDC
- **Shared drawing using DMACplusLCDC**: Rendering between Media Engine to a direct framebuffer attached to the LCD.

### CSC (Color Space Conversion)
- CSC VME: YCbCr data to RGB using the VME. YCbCr source planes are separated (Y1/Cb1/Cr1)
- CSC AVC: The AVC hardware module requires a Y plane and a packed CbCr plane. YCbCr data needs to be organized in a specific band-based format:
  - 16-pixel-wide vertical bands: Data must be chunked into these narrow vertical strips
  - Alternating pattern: Within 32-pixel-wide blocks, bands alternate between left and right halves
    - First 16 pixels go to the left half
    - Next 16 pixels go to the right half
    - Pattern continues across the image width

### VME (Virtual Mobile Engine / DSP)

Please see [Cracking The Unknown Project](https://github.com/mcidclan/psp-media-engine-cracking-the-unknown)

### The 0xBCC00000 Registers
I noticed that the activation of the ME bus clock (bit 0 of the HW register `0xBC100050`) appears to be related to hardware that is able to process over the local eDRAM and main VRAM. By applying a specific configuration, it seems to disable it or the local eDRAM.

This hardware seems to be involved at different moments during system execution. By default, the Media Engine's core initializes it with certain parameters that cause this component to autonomously apply background processes in the local eDRAM during homebrew usage. By observing the eDRAM, we can see a slow but progressive evolution of its content at certain memory ranges.

To avoid this situation, you can configure its hardware registers like so:
```c
hw(0xBCC00000) = -1;
hw(0xBCC00030) = 1;
hw(0xBCC00040) = 1;
```
This will prevent it from processing over the eDRAM, which also appears to improve the performance of code running on the Media Engine. If you do not use the eDRAM, just clear bit 0 of the bus clock register.

While the first hardware register appears to be a configuration register, the others seem to be related to frequencies, but this needs more investigation to clarify their exact meaning.

These hardware registers are probably related to the VME itself or the H.264 decoder, which would be more coherent since changing these configurations does not affect VME DSP processes.

## Utils

### Synchronization & Interrupts
- **Spinlock**: Hardware Register that can be used as a mutex between main CPU and the Media Engine. The spinlock uses a hardware register (0xbc100048) in which the main CPU can only modify bit[0] and the media Engine can only modify bit[1].

- **Software interrupts**:  To achieve this, it is necessary to enable system-level interrupts for the Media Engine and external interrupts at its CP0 level, via the IM mask above the Status register. An exception handler must be configured.

### Considering Memory Access
- **Cached memory**: To avoid cache line conflicts on shared memory, variables can be aligned to 64 bytes. This is important to consider when uncached and cached accesses occur on the same variable. In general, local implementation of cache functions needs to be done, as this allows local control over memory and cache updates and cleanup

- **Uncached memory**: To avoid cache line conflicts, a simple approach is to disable caching for shared memory using 0x40000000

## PSP Media Engine Custom Core Project

A project to map the PSP's Media Engine core functions with the purpose of using them in our code. The idea is simply to make the native kernel/core functions of the media engine available to homebrew developers.  
See: [https://github.com/mcidclan/psp-media-engine-custom-core](https://github.com/mcidclan/psp-media-engine-custom-core)  

## PSP Media Engine Samples / Examples

This part gathers links to recent samples and code examples related to the PSP Media Engine. Backed by older community efforts, these samples are the result of personal exploration, trial and error, some hardware-level investigation, and occasional reverse engineering when needed. The approach has been mostly empirical, driven by experimentation and curiosity.

The goal is to share findings and references that might be useful to others interested in working with the PSP's Media Engine.

- **Transfering data between main memory and the vme/dsp internal 24-bit buffers**  
  [https://github.com/mcidclan/vme-primary-dmac](https://github.com/mcidclan/vme-primary-dmac)  

- **Minimal reset handler**  
  [https://github.com/mcidclan/me-minimal-handler](https://github.com/mcidclan/me-minimal-handler)  

- **Hardware spinlock as a mutex between both CPUs**  
  [https://github.com/mcidclan/me-spinlock](https://github.com/mcidclan/me-spinlock)  

- **Using DMACPlus for transferring data between both CPUs**  
  [https://github.com/mcidclan/me-dmacplus-transfer](https://github.com/mcidclan/me-dmacplus-transfer)  
  [https://github.com/mcidclan/me-dmacplus-sc2me-then-me2sc](https://github.com/mcidclan/me-dmacplus-sc2me-then-me2sc)  
  [https://github.com/mcidclan/me-dmacplus-me2sc](https://github.com/mcidclan/me-dmacplus-me2sc)  

- **Simple counter using shared uncached memory**  
  [https://github.com/mcidclan/me-simple-count](https://github.com/mcidclan/me-simple-count)  

- **Writing to the same displayed framebuffer from both CPUs**  
  [https://github.com/mcidclan/me-sharing-drawing](https://github.com/mcidclan/me-sharing-drawing)  

- **Sending software interrupts between both CPUs**  
  [https://github.com/mcidclan/me-soft-interrupt](https://github.com/mcidclan/me-soft-interrupt)  

- **Requesting the Media Engine-hosted AVC hardware module to perform hardware colorspace conversion.**  
  [https://github.com/mcidclan/me-avc-csc-hd](https://github.com/mcidclan/me-avc-csc-hd)

- **Requesting the VME to perform hardware colorspace conversion**  
  [https://github.com/mcidclan/me-vme-csc-hd-992x992](https://github.com/mcidclan/me-vme-csc-hd-992x992)  
  [https://github.com/mcidclan/me-vme-hardware-csc](https://github.com/mcidclan/me-vme-hardware-csc)  

- **Writing and updating a GE list concurrently from both CPUs**  
  [https://github.com/mcidclan/me-feed-graphics-engine](https://github.com/mcidclan/me-feed-graphics-engine)  

- **Just a tiny lib**  
  [https://github.com/mcidclan/me-tiny-lib](https://github.com/mcidclan/me-tiny-lib)  

## Disclaimer
This information and related code are provided as-is without warranty. Users assume full responsibility for any implementation or consequences. Use at your own discretion and risk

## Related work

[PSP Media Engine Safe Task](https://github.com/mcidclan/psp-media-engine-safe-task)  
[PSP Media Engine Custom Core](https://github.com/mcidclan/psp-media-engine-custom-core)  
[PSP Media Engine Cracking The Unknown](https://github.com/mcidclan/psp-media-engine-cracking-the-unknown)  

## Thanks

Thanks to:
- crazyc, hlide, CelestBlue, artart78, zecoxao, davee, xerpi, John (Hedge), Kotcrab
- All contributors to the PSP Hardware Register's page
- Anyone else who contributed through documentation or reverse engineering

*m-c/d*
