# Media Engine Reload: What's New from 2025

## Code Execution Setup

Code execution on the Media Engine is achieved through the reset vector in kernel mode. This requires kernel-level privileges through one of three approaches:

1. **External kernel module**: Code deployed within a kernel module loaded from userland
2. **Kernel bridge module**: Small kernel module containing a bridge to handle callbacks
3. **Kexploit**: Eliminates external PRX dependency but increases boot time

The initialization code must be copied from System Control to the reset vector, which is located at address `0xBFC00000`. The Status Register requires proper configuration to clear exceptions and enable the FPU before running your code.

## Memory Map

### 0x00000000: EDRAM
- **PSP-1000**: 2MB internal EDRAM  
- **PSP-2000+**: 4MB internal EDRAM
- **Usage**:
 - Can be used for Color Space Conversion (CSC) processing via CSC controller
 - Source/destination for DMACplus transfers
 - Compatible with internal DSP-fed DMAC
 - General-purpose local memory for custom processing

### 0x04000000: Ping-Pong Buffers
Activated by setting bit 5 (DMA A bus) of hardware register `0xBC100050` (Media Engine side).

- **Main buffers**: 4 × 8KB 24-bit buffers (32-bit word aligned)  
 Range: `0x04000000` to `0x04008000`
- **Extended buffers**: Additional 24-bit buffers (potential mirrors, requires more testing)  
 Range: `0x04008000` to `0x04088000`

### 0x040F8000: DMAC Multiplexer
Complex DMAC handling multiple sources/destinations across 24-bit buffers and unidentified inputs (likely audio FIFO outputs). Activated with DMA A bit. Configuration complexity stems from unclear bit significance in 4 main hardware config registers.

### 0x040FF000: Primary DMAC
Handles data transfers between ping-pong buffers and main memory from Media Engine. Shares hardware control registers with Multiplexer DMAC but uses different configuration parameters. Additional sources/destinations require further testing.

### 0x04100000-0x04200000: DSP-fed DMAC (DMAC B)
Activated by setting bit 6 (DMA B bus) of hardware register `0xBC100050` (Media Engine side). Less complex than Multiplexer DMAC with apparent data transformation capabilities. Requires testing to identify potential DSP filters (FFT, FIR, IIR).

## Components

### DMACplus
DMACplus provides bidirectional DMA transfers between Media Engine and Main CPU, managed by the main system control.

### DMACplusLCDC
- **Shared drawing using DMACplus + LCDC**: Rendering between Media Engine to a direct framebuffer attached to the LCD.

### Processing Units
- **CSC (Color Space Conversion)**
 - CSC AVC:
 - CSC VME:
- **VME Virtual Mobile Engine / DSP**:

## Utils

### Synchronization & Interrupts
- **Spinlock**: Hardware Register that can be used as a mutex between main CPU and the Media Engine.
- **Software interrupts**:

### Considering Memory Access
- **Cached memory**: 
- **Uncached memory**:


## PSP Media Engine Samples / Examples

This part gathers links to recent samples and code examples related to the PSP Media Engine. These samples are the result of personal exploration, trial and error, some hardware-level investigation, and occasional reverse engineering when needed. The approach has been mostly empirical, driven by experimentation and curiosity.

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


*m-c/d*
