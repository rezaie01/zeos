**GCC (GNU Compiler Collection)** ist das Hauptwerkzeug zum Übersetzen von Quellcode in ausführbare Programme.
**Bestandteile:** Compiler für C, C++, Fortran usw., Preprocessor, Assembler, Linker-Unterstützung, Bibliotheken (`libgcc`, `libstdc++`).

In LFS wird es in zwei Phasen gebaut:
1. **[[GCC - Pass 1]]:** temporärer Cross-Compiler im Host-System (`$LFS/tools`)
2. **[[GCC - Pass 2]]:** finaler Compiler im neuen LFS-System.  
    Der `sed`-Befehl ändert eine GCC-Datei, damit er auf 64-Bit-Systemen Bibliotheken in `/lib` statt `/lib64` sucht — LFS verwendet nur `/lib`, um das System einfacher und konsistenter zu halten.