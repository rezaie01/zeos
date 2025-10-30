# Linux Standard Base (LSB)

## 🎯 Kernziele
- **Binärkompatibilität** zwischen Distributionen
- Standardisiert: 
  - [[FHS]]-Struktur
  - CLI-Tools (`ls`, `grep`, `tar`)
  - Paketformate (.rpm/.deb)

## 🔧 Schlüsselkomponenten
1. **LSB Core**  
   - Mindestanforderungen: Kernel, glibc, Basisbefehle
   - Init-Systeme (SysV → [[systemd]])

2. **LSB Libraries**  
   - Libc, Libm, Libpthread

3. **LSB CLI**  
   - Pfadstandards: `/bin`, `/usr/sbin`

## 📅 Versionen & Status
| Version | Highlights | Status |
|---------|------------|--------|
| 5.0 | Letzte Major-Version | 2015 eingestellt |
| 4.0 | Desktop-Integration | Deprecated |
| 3.0 | 64-Bit-Support | Legacy |

## ⚠️ Ablösung durch
- **systemd** (Init-System)
- **Freedesktop.org** (GUI)
- Distro-spezifische Standards

---
**Siehe auch**: [[FHS]], [[systemd]], [[Linux-Paketmanagement]]