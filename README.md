# vmp-lift

Handler walker / VIP tracer for **unpacked** VMProtect 3.8–3.10+ x64.

[![image.png](https://i.postimg.cc/h4ymnzJW/image.png)](https://postimg.cc/vDnTLmL3)


## Why 3.8 / 3.10 suck for older tools

Stuff aimed at ~3.5 (NoVmp etc.) assumed a pretty stable world: read a VIP byte, hit a dispatch stub, one op, repeat. Opcode tables mostly worked.

Then 3.7+ released, and 3.8–3.10 lean into it:

- **Merged handlers** — one native chunk often does several VM ops *and* the next-handler math. Old per-opcode maps just guess wrong.
- **Rolling key** — VIP immediates are encrypted. Decrypt is `enc ^ key`, then a per-handler mix (add/neg/rol/…), then `key ^= decrypted`.
- **Everything moves** — VIP / VSP / key sit in random GPRs. Sections look like `.kbB0` / `.WhW0` instead of a nice `.vmp0`.
- **Weird enters** — a lot of the research samples are `push enc; call enter_stub` (sometimes via a `.text` jmp), not “PE entry = vmenter.”

So this is emu-first on purpose. Walk from a real vmenter, peel what Unicorn actually does (VIP reads, VSP, ALU), dump that, and if it's a dumb pure function (`rcx,rdx → rax`) try to synth it into `devirt_l4.c`. Static mnemonic classify is only a backup.


## Usage

```
build\Release\vmp-lift.exe scan ..\samples\adder.vmp.exe
build\Release\vmp-lift.exe lift ..\samples\adder.vmp.exe
build\Release\vmp-lift.exe lift ..\samples\WaveUIAuth_static_unpacked.dll --vmenter 0x18101da77
```


## Output

Whatever you pass as `--out-dir`:

| file | |
|------|--|
| `report.txt` | quick summary |
| `lifted.ll` / `lifted.c` | handler IR |
| `vip_trace.*` | emu bytecode stream |
| `devirt.vasm` / `devirt_native.c` | peeled ops → stack C |
| `devirt_l4.c` | closed form when probes work (adder comes out as `a+b`) |
| `summary.json` | counts / flags |


## Refs

Recon 2024 VMP 3.8 talk, vxcall / r0da posts, [mzakocs samples](https://github.com/mzakocs/VirtualizationObfuscatorAnalysis).
