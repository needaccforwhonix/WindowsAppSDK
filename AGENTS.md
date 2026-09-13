# Windows App SDK – AI Contributor Guide

Comprehensive guidance for AI contributions to the Windows App SDK.

## Quick Reference

- **Build**: Use `BuildAll.ps1` or `BuildAll.cmd` at repo root (builds the full SDK solution)
- **Run**: Deploy and test on Desktop (x64, x86, ARM64) in Debug and Release configurations
- **Verify**: Run tests with `TestAll.ps1` or `TestAll.cmd`
- **Exit code 0 = success** – do not proceed if build fails

### Build Examples
```powershell
./BuildAll.ps1                                       # Build all SDK components
./BuildAll.ps1 -Platform x64 -Configuration Release # Specific platform and config
msbuild WindowsAppRuntime.sln /p:Configuration=Debug /p:Platform=x64
```

## Key Rules

- Code should be **production-quality** – follow SDK coding standards and best practices
- Follow **modular design** – each component should have clear boundaries and responsibilities
- Support all platforms: x64, x86, ARM64 in both Debug and Release
- Set minimum supported OS version to Windows 10 version 1809 (build 17763)
- Include copyright headers in all source files
- Build clean with no warnings or errors
- Ensure all tests pass before submitting changes

## Project Structure

The SDK follows this organization:
```
\dev                    # Source code for SDK components
     \<ComponentName>   # Individual SDK component
\build                  # Build scripts and pipeline definitions
\test                   # Test projects
\tools                  # Development tools and utilities
\specs                  # Feature specifications
\docs                   # Documentation
```

- **ComponentName**: SDK component (e.g., `AppLifecycle`, `DynamicDependency`, `MRTCore`)
- **Language**: Primarily C++ (C++/WinRT) with some C# projections
- **Tests**: Located under `\test` with component-specific test projects

## Style Enforcement

- **C#**: Follow `.editorconfig` at repo root (4-space indent, Allman braces, UTF-8 BOM)
- **C++**: Use C++/WinRT; follow `docs/Coding-Guidelines.md`
- **XAML**: Use consistent formatting

## Copyright Headers

For C#/C++/IDL files:
```csharp
// Copyright (c) Microsoft Corporation.
// Licensed under the MIT License.
```

For XAML files:
```xml
<!-- Copyright (c) Microsoft Corporation.
     Licensed under the MIT License. -->
```

## When to Ask for Clarification

- Ambiguous requirements after reviewing docs
- Cross-feature impact unclear
- API usage patterns not well documented

## Detailed Documentation

- [Coding Guidelines](docs/Coding-Guidelines.md) – Complete coding standards and checklist
- [Contributor Guide](docs/contributor-guide.md) – Contribution requirements
- [PR Template](docs/pull_request_template.md) – Pull request checklist
- [Windows App SDK Docs](https://docs.microsoft.com/windows/apps/windows-app-sdk/) – Official documentation



## 🛡️ Strict Embedding Separation, Zero-Fallback Law & 24/7 Dual-GPU Invariant
- **Reference**: `/home/m1st/.agents/rules/RULE_Strict_Embedding_Separation_And_Dual_Pipeline.md`

### 1. Das Absolute Fallback-Verbot (Zero-Fallback Law)
Unter keinen Umständen, zu keinem Zeitpunkt und aus keinem Grund darf ein Fallback zwischen verschiedenen Embedding-Modellen stattfinden.
* **Geltende Aktion:** Fällt ein Embedding-Modell aus oder ist überlastet, MUSS die Operation sofort hart fehlschlagen (`Fail-Fast`) oder die Payload transaktional in einer Queue (NATS/SQLite) verharren, bis das exakte Modell bereit ist.
* **Verboten:** Kein stiller oder dynamischer Modellwechsel (weder Jina -> Gemma noch umgekehrt).

### 2. Warum ein Embedding-Fallback mathematisch & informationstheoretisch unmöglich ist
* **Topologische Inkompatibilität heterogener Vektorräume (Non-Isomorphism):**
  Jedes Modell $f_\theta: \mathcal{X} \to \mathbb{R}^D$ projiziert Text in eine spezifische, gelernte Riemannsche Mannigfaltigkeit. Jina v5 ($D=256$) und EmbeddingGemma ($D=768$) spannen zwei völlig inkompatible geometrische Räume auf. Die Basisvektoren der semantischen Achsen sind ohne explizite Procrustes-Transformation nicht ausgerichtet.
* **Kollaps der Kosinus-Ähnlichkeit ($	ext{sim} \approx 0$):**
  Wird eine Suchanfrage mit Modell $B$ berechnet ($v_q = f_B(q)$), während der Dokumentenkorpus mit Modell $A$ indiziert wurde ($v_d = f_A(d)$), verhält sich das Skalarprodukt mathematisch wie das zweier rein zufälliger Vektoren auf einer hochdimensionalen Einheitssphäre:
  $$\mathbb{E}[\text{sim}(u, v)] = 0 \quad \text{mit Varianz} \quad \sigma^2 = \frac{1}{D}$$
  Der Nearest-Neighbor-Algorithmus (HNSW/k-NN) liefert stochastisches Rauschen. Das RAG-System erhält völlig falsche oder irrelevante Kontexte.
* **Irreversible Index-Vergiftung (Index Poisoning):**
  Wird auch nur ein einziger Vektor von Modell $B$ als "Fallback" in den Index von Modell $A$ geschrieben, verunreinigt er die Distanzgraphen und Clusterzentren dauerhaft.
* **Das Gesetz des Fail-Fast:**
  Ein Ausfall muss hart abbrechen (`HTTP 503 Service Unavailable / IngestionQueueBlocked`).

### 3. Duale 24/7 Erfassungspflicht (GPU-Only)
* **GPU-Only Mandat:** Es läuft absolut nichts auf der CPU — GPU ONLY (NVIDIA GB10 CUDA) für ausnahmslos jedes Embedding-Modell.
* **24/7 Parallelität:** Sowohl `jina-embeddings-v5-omni-nano-classification` (256D, ~4,1 GB VRAM) als auch `google/embeddinggemma-300m` (768D, ~1,2 GB VRAM) laufen dauerhaft 24/7 im VRAM (Summe ~5,3 GB VRAM).
* **Duale Erfassung:** Jeder zu indizierende Text/Chunk wird immer von beiden Modellen parallel eingebettet und getrennt persistiert.

### 4. Idioten- & Failsafe-Sicherung auf Datenbankebene
* **SQLite Schema CHECK-Constraints:**
  `model_signature TEXT NOT NULL CHECK(model_signature = '...')` und `dimension INTEGER NOT NULL CHECK(dimension = ...)` erzwingen atomare Abbrüche auf Engine-Ebene bei Modell-Mismatches.
* **Qdrant Collection Constraints:**
  Strikte Trennung in separate Collections (`dgx_text_embeddings_jina_256` vs `dgx_text_embeddings_gemma_768`) mit fixierter Vektordimension.


## ⚡ High-Quality Systems Programming Languages Priority (No-Python Policy)
- **Reference**: `/home/m1st/.agents/rules/RULE_High_Quality_Systems_Programming_Languages.md`
- **Rule**:
  1. **Bevorzugte Sprachen:** High Quality **Golang (Go), Rust, C++, Zig, PowerShell, C** sind IMMER und AUSNAHMSLOS die bevorzugten Programmiersprachen.
  2. **Kein Python:** Python ist für neue Daemons, Watcher, Automatisierungen, APIs, CLI-Tools und Dienste strikt untersagt (GIL-Bottlenecks, Dependency-Drift, Speicherineffizienz).
  3. **Natives Systems-Engineering:** Alle Hintergrunddienste, Caching-Ebenen und Task-Runner müssen als native, speichersichere und nebenläufige Binaries kompiliert werden.
