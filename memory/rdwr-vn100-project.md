---
name: rdwr-vn100-project
description: State and key findings for the rdwr_vn100 VN-100 IMU tool and the fc.py 200 Hz goal
metadata: 
  node_type: memory
  type: project
  originSessionId: aa4bfd5d-c6b3-4f32-ac00-4dca6460bfe2
---

`rdwr_vn100` is a Rust CLI (in /home/wink/data/prgs/nps-gnc/rdwr_vn100) the user built to read/configure a VectorNav **VN-100** IMU over serial (`/dev/ttyUSB0`, an **FTDI FT232R**). Subcommands: `get`, `set <hz> [--persist]`, `baud <n> [--persist]`, `reset`, `factory-reset`, `bench [--hz][--secs]`, `help`. VN ASCII protocol `$VN...*XX` (XOR checksum); binary output uses `vn_crc16` (CCITT). 22 unit tests pass.

**Real goal:** get the user's flight controller `../fc/src/fc.py` (symlink → `fc-current.py`) to read the IMU at **200 Hz** instead of the 40 Hz it currently gets.

**Root cause of the 40 Hz (confirmed):** `VecNavHandler` uses the vendor SDK's `Sensor.autoConnect()` (probes [115200, 921600, …], finds device at its default 115200) and **never writes the device output rate** — `rate=200`/`baudrate=921600` params are effectively no-ops. So the device streams its default 40 Hz VNYMR. 40 Hz = VN-100 factory default async rate.

**Why 200 Hz failed at 115200:** it's a *bandwidth* limit, not a frequency one. Default VNYMR ASCII ≈115 B; 200 Hz × 115 B ≈ 230 kbit/s > the ~115 kbit/s link → device returns `$VNERR,0C` (InsufficientBaudRate). Same reason `set 100` failed but 40/50 worked.

**USE 115200 — high baud is intermittently fatal (refined twice, 2026-06-21):** on this FT232R + cable, a cross-process reconnect at high baud can **wedge the VN-100's UART completely** — silent at *every* baud (incl. 115200) until a **power cycle**. The failure is **PROBABILISTIC per reconnect with NO clean threshold** — the odds rise with baud but there's no crisp cutoff. Observed: **115200 = zero wedges across many clean reconnects** (its only "failures" were baud-mismatch or already-wedged collateral); **230400 and even 921600 = intermittent** — sometimes wedge on the first reconnect, sometimes survive several (921600 survived 2 reconnects after a fresh power cycle at 16:48). A power cycle (clean device state) seems to lower the odds. NOTE: I twice over-cleaned this into false thresholds ("≥230400 deterministic", then "921600 ~always") and the user disproved both — it's all noisy/probabilistic. Mechanism: FTDI DTR/RTS-on-open transient, mis-framed more often as baud climbs. **Flight-safety logic:** an intermittent *unrecoverable* failure that PASSES bench testing is the worst kind — it ships, then bricks the IMU mid-flight (no recovery but cutting power). "Works sometimes" ≠ safe. 115200 is the only rate with a clean record and binary already gives 200 Hz there. To actually trust a higher baud you'd need a real reliability test (hundreds of reconnects, count wedges), not a few runs. Don't bother — stay at 115200.

**CHOSEN PATH (proven):** stay at rock-solid **115200**, switch the device to a **compact binary output** (reg 75: Common group = TimeStartup + Accel, `rateDivisor=4` → 800/4 = 200 Hz, 26 B/frame). `rdwr_vn100 bench` measured **exactly 200.0 Hz** (1000 frames/5 s, all CRC-valid, ~52 kbit/s of 1152). `fc.py` mostly needs Accel Y (launch detection); accel Y read ≈ fc.py's value, so same channel.

**DIRECTION CHANGE (2026-06-21):** the user will **probably NOT modify `fc.py`**. Instead the likely plan is a new project **`fcbr`** = "flight controller, binary, Rust" — reimplement the flight-controller read path in Rust using the VN-100 **binary** output (building on `rdwr_vn100`'s proven binary/CRC code) rather than the Python SDK. So the `fc.py` patch is on the back burner; see [[fc-edit-requires-permission]].

Still worth doing when fresh: **independent verification** of the 200 Hz (vendor SDK + raw pyserial), per [[user-wants-independent-verification]].

A `README.md` documenting all of this was written into the rdwr_vn100 repo on 2026-06-21. The device was restored to 115200 / 40 Hz ASCII (bench cleans up after itself).
