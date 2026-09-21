# 量子運算研究方向 閱讀與實作紀錄

**姓名**: 何彥霆  
**指導教授**: 涂嘉恒 老師 (ASRLab)  
**開始日期**: 2026-09-04  
**目前研究方向**: Compiler Toolchains for Quantum Computing

---

## 進度總覽

| 日期 | 項目 | 狀態 | 備註 |
| --- | --- | --- | --- |
| 2026-09-04 | 收到閱讀與實作清單 | ✅ | 建立 HackMD，開始整理研究方向 |
| 2026-09-06 | Quantum computing basics | ⏳ | 確認需要掌握的範圍 |
| 2026-09-07 ~ 09-15 | 離職、開學與搬遷相關事務 | ✅ | 筆記進度暫緩；利用零碎時間初步閱讀 XACC / QAOA 相關資料 |
| 2026-09-15 | Quantum computing basics | ✅ | 建立基本概念，不深入 physical implementation |
| 2026-09-16 | Quantum toolchains background | ✅ | 整理 XACC、qcor、QIR、CUDA-Q 的基本發展脈絡 |
| 2026-09-16 ~ 09-17 | XACC system paper | ✅ | 閱讀 system-level design、IR、Accelerator abstraction |
| 2026-09-17 ~ 09-18 | qcor paper | ✅ | 延續 XACC，理解 single-source C++、Clang plugin 與 compiler workflow |
| 2026-09-18 | Quantum-HPC / QAOA | ✅ | 快速了解 QAOA 與相關 workload，建立 hybrid quantum-classical workflow 的基本概念 |
| 2026-09-18 ~ 09-21 | CUDA-Q / Max-Cut with QAOA | ✅ | 建立 CUDA-Q 環境，完成 Quick Start，並使用 GPU simulator 跑通 Max-Cut / QAOA 範例 |
| 2026-09-18 ~ 09-21 | XACC / qcor Hands-on | ✅ | Ubuntu 20.04 source build XACC / qcor，處理 build 問題並完成 qpp simulator 測試 |
| 2026-09-21 | Questions & Reflections | ✅ | 根據 paper reading 與 hands-on 結果整理目前理解、問題與可能研究方向 |



---

## 1. 量子運算基礎觀念

### 1-1. Fundamental
**A classical bit is either 0 or 1, while a qubit can be in a superposition.**

Three fundamental concepts:

* Superposition : a quantum state can contain amplitudes for multiple basis states.
* Entanglement : multiple qubits may form a joint state that cannot be described independently.
* Interference : quantum operations can amplify or suppress probability amplitudes.


> **My understanding:**  
Quantum computers do not simply "compute all inputs simultsneously."
 Superposition allows multiple basis states to coexist, but measurement only
 returns one result according to |amplitude|^2.
>
> The useful part is **interference**: quantum algorithms arrange unitary
 operations so that incorrect amplitudes cancel (destructive interference)
 while desired amplitudes are amplified (constructive interference).

### 1-2. Quantum Computing Approaches (brief comparison)

**Computational Models：Gate Model vs. Adiabatic vs. Topological**

Quantum computing can be understood through several different computational models:

| Model | Basic Idea | Relevance |
| --- | --- | --- |
| **Gate model** | Computation is represented as sequences of unitary gates followed by measurement. | **Focus** — used by XACC/QCOR/CUDA-Q |
| **Adiabatic / Annealing** | Computation through evolution toward the ground state of a target Hamiltonian. | Brief understanding only |
| **Topological** | Encodes quantum information using topologically protected states. | Hardware/fault-tolerance approach; not explored further |

> **Scope:** I focus on the gate model because it is the computational model most directly related to the compiler toolchains studied below.


### 1-3. Physical qubit modalities (brief survey)
**Several hardware approaches currently compete for building qubits, each with different trade-offs:**

| Modality | Qubit carrier | Key trade-off |
| --- | --- | --- |
| Superconducting | Josephson junction circuits | Mature platform, but requires mK cooling |
| Trapped ion | Ion internal energy levels | High fidelity, slower to scale |
| Neutral atoms | Optical-tweezer-trapped atoms | Fast-scaling, flexible array reconfiguration |
| Quantum dot | Electron spin in semiconductor | Leverages existing fab processes |
| Linear optical | Photon polarization/path | Room-temperature, but weak inter-qubit coupling |
| Colour centre | Defect spin (e.g., NV center) | Room-temperature, limited scalability |

> **Scope:** Physical implementations are only surveyed briefly here.
> My focus is the software/compiler layer that abstracts different QPU backends.


### 1-4. Summary

For this study, I focus on **gate-based quantum computing and its
software/compiler abstractions**.

Physical qubit implementations and other computational models are kept at
a conceptual level, since the main topic is heterogeneous
quantum-classical compiler toolchains.
 
---
 
 

## 2. 研究方向探索：Quantum-Classical Computing

先從兩個不同角度了解 quantum-classical computing：

- **2-1. High-Performance Computing with Quantum Computing**：從實際 hybrid workload 出發，了解 QAOA / Max-Cut，以及 problem size、quantum resource 與 scalability 等問題。
- **2-2. Compiler Toolchains for Quantum Computing**：從 system software 出發，了解 quantum program 如何經過 compiler、IR、runtime / middleware，最後執行於不同 quantum backend。

目前不急著直接決定具體研究題目，而是先透過閱讀與實作建立兩邊的基本理解，再思考後續比較適合深入的問題。

---

### 2-1. High-Performance Computing with Quantum Computing

### 2-1-1. Max-Cut with QAOA — CUDA Quantum

> **Example:** NVIDIA CUDA-Q — *Max-Cut with QAOA*  
> **閱讀目的:** 先理解一個實際的 hybrid quantum-classical workload，以及 classical / quantum computation 如何互相配合。

**Max-Cut** 是將 graph 的 vertices 分成兩組，使跨越兩組的 edges 數量最大化。

QAOA (Quantum Approximate Optimization Algorithm) 是一種 **hybrid quantum-classical algorithm**。

在 CUDA-Q 的範例中，我目前把 execution flow 理解成：

1. 將 Max-Cut problem encode 成 cost Hamiltonian。
2. 建立 parameterized QAOA quantum circuit。
3. Quantum circuit 根據目前 parameters 計算 expectation value。
4. Classical optimizer 根據結果更新 parameters。
5. 重複 quantum execution ↔ classical optimization。
6. 最後 sample optimized circuit，得到可能的 Max-Cut solution。

```text id="73wnrp"
Max-Cut
   ↓
Parameterized QAOA Circuit
   ↓
Quantum Execution
   ↓
Expectation Value
   ↓
Classical Optimizer
   ↓
Update Parameters
   └────────────→ repeat
```

因此 QAOA 並不是單純把整個問題交給 QPU，而是一個 classical optimizer 與 quantum execution 反覆互動的 workload。

目前先理解這個 execution model，不深入 QAOA 的數學推導；實際執行結果記錄於後面的 CUDA-Q hands-on。


---
 **Initial Understanding:**  
 
 #### Initial Understanding

 在實作前，我先建立整體概念，沒有深入 quantum algorithm 的數學推導。這部分主要讓我看到一個實際 hybrid workload 如何產生 scalability problem，以及 classical computation 如何參與 quantum workload。

---

### 2-2. Compiler Toolchains for Quantum Computing

### 2-2-1. Brief Background

Quantum-classical computing has gradually evolved toward treating **QPUs as accelerators** within heterogeneous computing systems. Some related developments are summarized below:

| Year | Project | Main Idea |
| --- | --- | --- |
| **2019** | **XACC** | A system-level, hardware-agnostic infrastructure that treats QPUs as accelerators for heterogeneous quantum-classical computing. [1] |
| **2020** | **QIR** | An LLVM-based intermediate representation designed as a common interface between quantum programming languages and target platforms. [2] |
| **2020** | **QCOR** | A C++ language extension and compiler for single-source quantum-classical programming, built upon XACC. [3] |
| **2022 → 2023** | **QODA → CUDA Quantum** | NVIDIA's platform for a unified hybrid quantum-classical programming model. QODA was renamed CUDA Quantum in 2023. [4] |

#### References

[1] A. J. McCaskey et al.,  
[*XACC: A System-Level Software Infrastructure for Heterogeneous Quantum-Classical Computing*](https://arxiv.org/abs/1911.02452), 2019.

[2] A. Geller,  
[*Introducing Quantum Intermediate Representation (QIR)*](https://devblogs.microsoft.com/qsharp/page/8/), Microsoft Q# Blog, Sep. 23, 2020.

[3] T. Nguyen et al.,  
[*Extending C++ for Heterogeneous Quantum-Classical Computing*](https://arxiv.org/abs/2010.03935), 2020.

[4] NVIDIA,  
[*NVIDIA Announces Hybrid Quantum-Classical Computing Platform*](https://nvidianews.nvidia.com/news/nvidia-announces-hybrid-quantum-classical-computing-platform), Jul. 12, 2022.  
QODA was renamed CUDA Quantum on Mar. 21, 2023.

---

### 2-2-2. XACC — A System-Level Software Infrastructure for Heterogeneous Quantum-Classical Computing

> **Paper:** *XACC: A System-Level Software Infrastructure for Heterogeneous Quantum-Classical Computing* (2020)  
> **定位:** System-level quantum programming framework  
> **閱讀目的:** 了解 heterogeneous quantum-classical computing 的軟體架構，以及 frontend、IR、transformation 與 backend 如何透過統一 framework 串接。

#### Why start with XACC?

XACC provides the system-level infrastructure underlying qcor and introduces the idea of integrating QPUs as accelerators into heterogeneous computing environments.

Understanding this abstraction first should make the later compiler-level design in qcor easier to follow.

#### 閱讀目標

一開始看到 XACC 時，我還不太清楚它和一般 quantum programming framework 的差別，因此沒有打算把整篇論文的所有 interface 都弄懂，而是先想了解：

- XACC 為什麼被提出？
- 它在 quantum-classical toolchain 中負責什麼？
- 一個 quantum program 大概會怎麼從 XACC 到 QPU？

因此主要閱讀 **Introduction、Features of XACC、Framework Architecture**，後面的 interfaces 則先挑和 compiler / execution flow 比較相關的部分閱讀。

---

#### QPU as an Accelerator

原本我對 quantum software 的想像比較簡單：

```text id="t3vldj"
Quantum Program
      ↓
Quantum Framework
      ↓
Remote QPU / Simulator
```

但 Introduction 提到，當時常見的方式多半是透過 high-level framework 和 remote API 使用 QPU，而 XACC 想處理的是未來 **CPU 與 QPU 更緊密整合**之後的問題。

XACC 把 QPU 看成 classical computer 的 **accelerator / co-processor**：

```text id="n63s8b"
Classical Application
        │
        │ offload quantum kernel
        ▼
       QPU
        │
        │ result
        ▼
Classical Application
```

這個概念和 CPU + GPU 的 heterogeneous computing 有些相似。

> **我的理解：**  
> XACC 並不是想取代 classical computation，而是提供一個 infrastructure，讓 classical program 可以把部分工作交給 QPU，再取得結果繼續執行。

---

#### Why System-Level?

這是我讀 Introduction 和 Features 時比較有興趣的地方。

論文認為透過 remote API 操作 QPU 的方式，未來可能逐漸變成更緊密的 CPU–QPU integration，因此需要比 high-level quantum framework 更低階的 system software。

文中提到未來可能從：

```text id="u09hld"
remote process invocation
```

走向：

```text id="8n6nkk"
in-process / in-memory device driver access
```

目前我還不清楚真正的 CPU–QPU driver 或 communication mechanism 會長什麼樣子，但這讓我開始理解為什麼作者把 XACC 定位成 **system-level software infrastructure**。

---

#### XACC Toolchain and IR

接著我主要看 Framework Architecture，而沒有深入每一個 interface。

```text id="bx2rpo"
Front-end
    ↓
Middle-end
    ↓
Back-end
```

我目前把 execution path 簡化理解成：

```text id="81om2e"
Quantum Source
(OpenQASM / Quil / XASM / ...)
        ↓
     Compiler
        ↓
      XACC IR
        ↓
 IR Transformation
        ↓
    Accelerator
        ↓
   QPU / Simulator
```

其中我覺得最重要的是 **IR**。

不同 quantum language 可以先被轉換成共同的 XACC IR，使後續 transformation 與 backend 不需要直接依賴原始語言。

```text id="k1j60c"
Without IR:
N languages × M devices → N × M mappings

With IR:
N languages → IR → M devices
             → N + M mappings
```

這讓我第一次比較具體理解 **IR 在 quantum toolchain 中為什麼重要**。

讀到這裡後，我也開始接觸 qcor 與後來出現的 QIR，並把：

```text id="44rnb7"
XACC IR → qcor → QIR / LLVM-based quantum toolchain
```

先作為後續可以繼續追的學習方向。

---

#### Accelerator and Hardware Abstraction

不同 QPU backend 的實作差異很大，因此另一個我想了解的是 XACC 如何做到 cross-platform。

目前我的理解是 XACC 定義了一個共同的 `Accelerator` interface：

```text id="dvhd4u"
              ┌─ IBM
              ├─ Rigetti
XACC ─────────┼─ IonQ
Accelerator   ├─ D-Wave
              └─ Simulator
```

上層只透過共同 interface 執行 quantum program，而 hardware-specific implementation 留給不同 backend。

因此：

> **Hardware-agnostic 並不是讓不同硬體的差異消失，而是透過 abstraction 將差異隔離在 backend。**

---

#### Initial Understanding

目前我會把 XACC 理解成：

> XACC 是一個以 C++ 為主的 quantum-classical system software framework。它把 QPU 視為 classical system 的 accelerator，並透過 Compiler、IR 和 Accelerator 等 abstraction，把 quantum language、compiler transformation 和實際 QPU backend 分開。

這篇主要讓我建立：

```text id="tz3jce"
Language
   ↓
Compiler
   ↓
IR
   ↓
Transformation
   ↓
Accelerator Abstraction
   ↓
QPU / Simulator
```

這條 system-level 路徑。

---

### 2-2-3. qcor — Extending C++ for Heterogeneous Quantum-Classical Computing

> **Paper:** *Extending C++ for Heterogeneous Quantum-Classical Computing* (2020)  
> **基於:** XACC  
> **閱讀目的:** 了解 XACC 之上的 compiler / language layer，以及 C++ 如何整合 classical 與 quantum code。

#### Why I read this

上一篇 XACC 主要讓我了解 quantum-classical software stack 中 IR、compiler interface 與 backend abstraction 的概念。

因此這篇沒有打算完整閱讀，而是想繼續往上一層看：

> **如果 XACC 已經提供 IR 和 backend abstraction，那 programmer 寫的 C++ quantum code 是怎麼進入 XACC 的？**

主要閱讀：

- Introduction
- Anatomy of a qcor Program
- Clang Plugins
- Compiler / Syntax Handler
- Compiler Workflow

其他 quantum algorithm、optimization benchmark 與 application examples 先快速瀏覽。

---

#### Single-Source C++ Programming

XACC 已經提供一套 hardware-agnostic quantum-classical programming framework，但主要採用 **dual-source programming model**。

qcor 想再往上一層，讓 classical code 和 quantum code 可以直接寫在同一份 C++ source code：

```cpp id="dkxzdb"
__qpu__ void bell(qreg q) {
    H(q[0]);
    CX(q[0], q[1]);

    for (int i = 0; i < 2; i++) {
        Measure(q[i]);
    }
}

int main() {
    auto q = qalloc(2);
    bell(q);
    q.print();
}
```

因此我目前把兩者的關係理解成：

```text id="u1h8tf"
C++ Application
      │
      │ __qpu__ quantum kernel
      ▼
     qcor
(Language Extension / Compiler)
      │
      ▼
    XACC IR
      │
      ▼
XACC Accelerator
      │
      ▼
QPU / Simulator
```

> **我的理解：**  
> XACC 解決的是「如何用統一 abstraction 表示、轉換並執行 quantum program」，而 qcor 更進一步處理「programmer 如何直接在 C++ 中表達 heterogeneous quantum-classical program」。

---

#### Compiler Workflow

原本我以為這可能需要直接修改 LLVM / Clang 本身，讓 compiler 認識 quantum instructions。

但 qcor 的做法不是直接修改 Clang，而是透過 **Clang plugin / SyntaxHandler** 處理 `__qpu__` quantum kernel，把其中的 quantum DSL 轉換成合法的 C++ / QuantumRuntime API calls，再交回原本的 Clang compilation flow。

目前我先把它簡化理解成：

```text id="uuh22h"
C++ + __qpu__ kernel
        ↓
 QCORSyntaxHandler
        ↓
   TokenCollector
        ↓
 QuantumRuntime API
        ↓
      XACC IR
        ↓
 XACC Accelerator
        ↓
   QPU / Simulator
```

其中 `TokenCollector` 讓不同 quantum language（例如 XASM、OpenQASM、Quil）可以進入相同的 runtime / XACC infrastructure。

這也讓我比較清楚兩者的分工：

- **XACC**：IR、transformation、backend 等 system-level abstraction。
- **qcor**：在 XACC 上加入 C++ language / compiler frontend。

---

#### Initial Understanding

這篇讓我第一次比較具體看到：

> **Quantum compiler 不一定代表重新建立一個完整 compiler。**

qcor 選擇利用既有的 Clang infrastructure，在 frontend 加入 extension，再將 quantum code 接到 XACC。

因此我開始想了解：

- 如果現在建立新的 quantum compiler / toolchain，哪些部分應該自己實作，哪些部分適合建立在 LLVM、MLIR 或其他既有 compiler infrastructure 上？
- 不同 quantum compiler 使用的 IR，在 representation、transformation 與 backend integration 上有什麼差異？

目前這些只先作為後續學習方向，沒有深入比較。

---






## 3. 實作

### 3-1. CUDA-Q / Max-Cut with QAOA

> **Goal:** 完成 CUDA-Q 基本環境建置，執行 Quick Start 範例，並按照官方 tutorial 跑通 Max-Cut with QAOA。

---

#### Environment

原本打算直接使用先前準備好的 **Ubuntu 20.04 (WSL2)**，先確認環境：

```bash
lsb_release -a
gcc --version
g++ --version
nvidia-smi
nvcc --version
```

當時環境：

- Ubuntu 20.04.6 LTS
- GCC / G++ 9.4.0
- RTX 3070 8 GB
- `nvcc` 尚未安裝

一開始 `nvidia-smi` 找不到，但確認是 WSL2 後，直接執行：

```bash
/usr/lib/wsl/lib/nvidia-smi
```

可以正常抓到：

```text
GPU: NVIDIA GeForce RTX 3070 8 GB
Driver Version: 591.86
CUDA Version: 13.1
```

因此 GPU passthrough 本身正常，只是 `/usr/lib/wsl/lib` 不在 `$PATH`。

另外，`nvidia-smi` 顯示的 `CUDA Version: 13.1` 只代表目前 driver 支援的 CUDA version，**不代表 WSL 中已安裝 CUDA Toolkit**，此時 `nvcc` 仍然不存在。

CUDA-Q Quick Start 建議使用支援 C++20 的 toolchain，而目前 Ubuntu 20.04 為 G++ 9.4。

考慮到下一個 XACC / QCOR 實作明確要求 Ubuntu 20.04，因此不修改原本環境，另外建立 Ubuntu 22.04：

```text
WSL2
├── Ubuntu 20.04 → XACC / QCOR
└── Ubuntu 22.04 → CUDA-Q / Max-Cut
```

建立 Ubuntu 22.04：

```powershell
wsl --install -d Ubuntu-22.04
```

安裝基本 C++ toolchain：

```bash
sudo apt update
sudo apt install build-essential
```

確認環境：

```text
Ubuntu 22.04.5 LTS
GCC / G++ 11.4.0
RTX 3070 detected
```

---

#### Installation & Setup

##### CUDA-Q

使用 CUDA 13 / x86_64 installer：

```bash
sudo -E bash install_cuda_quantum_cu13.x86_64 --accept
. /etc/profile
```

確認：

```bash
which nvq++
nvq++ --version
```

結果：

```text
/opt/nvidia/cudaq/bin/nvq++
nvq++ version 0.16.0
```

---

##### CUDA Toolkit

第一次執行 CUDA-Q C++ 程式時遇到：

```text
error while loading shared libraries:
libcublas.so.13: cannot open shared object file
```

雖然 WSL2 已經可以透過 Windows driver 使用 RTX 3070，但 Ubuntu 本身尚未安裝 CUDA Toolkit。

因此加入 NVIDIA WSL repository 並安裝 CUDA Toolkit 13.0：

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt install cuda-toolkit-13-0
```

設定環境：

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}' >> ~/.bashrc
source ~/.bashrc
```

確認：

```text
CUDA Toolkit: 13.0 (V13.0.88)
CUDA-Q: 0.16.0
```

> **Note:** `nvidia-smi` 顯示的 CUDA version 是 driver 所支援的版本，不代表 Linux 環境中已經安裝 CUDA Toolkit。

---

##### Python Environment

後面的 Max-Cut / QAOA 官方範例使用 Python，因此另外準備 Python CUDA-Q 環境。

Ubuntu 22.04 預設為 Python 3.10，直接安裝新版 `cudaq` 時失敗，因此不修改 system Python，改用 `uv` 建立 Python 3.12 virtual environment：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

uv python install 3.12

mkdir -p ~/cudaq-maxcut
cd ~/cudaq-maxcut

uv venv --python 3.12
source .venv/bin/activate

uv pip install cudaq
uv pip install networkx matplotlib
```

目前環境：

```text
Python: 3.12.14
CUDA-Q: 0.16.0
NetworkX: 3.6.1
```

這裡另外遇到一個問題：前面 C++ installer 已經在 `/opt/nvidia/cudaq` 安裝一套 CUDA-Q，而 Python wheel 又包含另一套 CUDA-Q libraries。

第一次執行：

```python
import cudaq
```

出現 MLIR `undefined symbol` error。

檢查後發現：

```text
CUDA_QUANTUM_PATH=/opt/nvidia/cudaq
LD_LIBRARY_PATH=...:/opt/nvidia/cudaq/lib
```

Python 因此混用了 C++ installation 的 shared libraries。

在 Python environment 中排除 C++ CUDA-Q：

```bash
unset CUDA_QUANTUM_PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64
```

之後即可正常載入：

```python
import cudaq

print(cudaq.__version__)
print(cudaq.get_target())
```

結果：

```text
CUDA-Q Version 0.16.0

Target: nvidia
simulator=custatevec_fp32
precision=fp32
```

代表 Python CUDA-Q 已可正常使用 NVIDIA / cuStateVec GPU simulator。

---

#### Quick Start Test

先按照官方 Quick Start 建立簡單的 CUDA-Q C++ 程式：

```cpp
#include <cudaq.h>

__qpu__ void kernel(int qubit_count) {
  cudaq::qvector qubits(qubit_count);
  h(qubits[0]);

  for (auto i = 1; i < qubit_count; ++i) {
    cx(qubits[0], qubits[i]);
  }

  mz(qubits);
}

int main(int argc, char *argv[]) {
  auto qubit_count = 1 < argc ? atoi(argv[1]) : 2;
  auto result = cudaq::sample(kernel, qubit_count);
  result.dump();
}
```

編譯並執行：

```bash
nvq++ program.cpp -o program.x
./program.x
```

結果：

```text
{ 00:528 11:472 }
```

1000 shots 中只出現 `00`、`11`，比例接近 1:1，符合 Bell state 的預期結果。

---

#### Max-Cut with QAOA

CUDA-Q Quick Start 確認可以正常執行後，接著按照官方 **Max-Cut with QAOA** tutorial 執行範例。

這次先以「成功跑完整個範例，並理解基本執行流程」為目標，不深入 QAOA 的數學推導。

##### Problem Setup

官方範例使用 5 個 nodes：

```python
nodes = [0, 1, 2, 3, 4]

edges = [
    [0, 1],
    [1, 2],
    [2, 3],
    [3, 0],
    [2, 4],
    [3, 4]
]
```

大致上的 graph：

```text
    1 -------- 2
    |          | \
    |          |  \
    0 -------- 3 -- 4
```

目標是把 nodes 分成兩組，讓跨越兩組的 edges 數量最多。

---

##### QAOA Circuit

按照官方範例建立 QAOA kernel：

```python
qubit_count = len(nodes)
layer_count = 2
parameter_count = 2 * layer_count
```

每個 graph node 對應一個 qubit。

目前先把 QAOA circuit 理解成：

```text
Initial superposition
        ↓
Problem layer
        ↓
Mixer layer
        ↓
重複數個 layers
```

其中 parameters 會交給後面的 classical optimizer 調整。

接著按照 graph 的 edges 建立 Max-Cut Hamiltonian，作為 optimization 時的 cost function。

---

##### Optimization

使用 CUDA-Q 的：

```python
cudaq.optimizers.NelderMead()
```

作為 classical optimizer，並透過：

```python
cudaq.observe(...)
```

取得目前 parameters 對應的 expectation value。

這次沒有按照 tutorial 改成：

```python
cudaq.set_target('qpp-cpu')
```

而是直接使用目前環境偵測到的 NVIDIA backend：

```text
Target: nvidia
Simulator: custatevec_fp32
GPU: RTX 3070
```

執行：

```bash
python maxcut.py
```

得到：

```text
optimal_expectation = -4.495972967939451
estimated max cut value = 4.495972967939451

optimal_parameters =
[0.513683034112723,
 -0.21346143987734223,
 0.32429923379972176,
 0.8876711395574044]
```

和官方 tutorial 的結果約 `-4.49597` 基本一致。

![Screenshot 2026-09-21 at 3.11.52 PM](https://hackmd.io/_uploads/ryWAqLRYfl.png)

---

##### Sampling

Optimization 完成後，再使用最佳 parameters 執行：

```python
counts = cudaq.sample(
    kernel_qaoa,
    qubit_count,
    layer_count,
    edges_src,
    edges_tgt,
    optimal_parameters
)
```

這次 sampling 中出現次數較高的結果包括：

```text
01010 : 148
01011 : 155
10100 : 144
10101 : 165
```

其中最高的是：

```text
Most probable bitstring = 10101
Count = 165
```

`10101` 是官方列出的 Max-Cut solutions 之一，對應的 cut value 為：

```text
5
```

因此 Max-Cut with QAOA 範例成功執行。

![Screenshot 2026-09-21 at 3.13.36 PM](https://hackmd.io/_uploads/HkWxiI0FMl.png)

---

#### Problems Encountered

| Problem | Cause | Solution |
| --- | --- | --- |
| `nvidia-smi` 找不到 | WSL NVIDIA tools 不在 `$PATH` | 使用 `/usr/lib/wsl/lib/nvidia-smi` 確認 GPU |
| `libcublas.so.13` missing | WSL 有 GPU driver，但尚未安裝 CUDA Toolkit | 安裝 CUDA Toolkit 13.0 |
| Python 3.10 無法直接安裝目前使用的 CUDA-Q | Python version 不符合套件需求 | 使用 `uv` 建立 Python 3.12 environment |
| MLIR `undefined symbol` | C++ installer 與 Python wheel 的 CUDA-Q libraries 混用 | 調整 `CUDA_QUANTUM_PATH` 與 `LD_LIBRARY_PATH` |

---

#### Result

- [x] CUDA-Q C++ environment
- [x] CUDA Toolkit / GPU environment
- [x] CUDA-Q Quick Start example
- [x] CUDA-Q Python environment
- [x] NVIDIA / cuStateVec backend
- [x] QAOA optimization
- [x] Sampling
- [x] 找到 Max-Cut solution `10101`
- [x] Max-Cut value = `5`

---

#### Current Understanding

這次實作先把 CUDA-Q 的基本 hybrid workflow 跑過一次：

```text
Graph
  ↓
QAOA quantum kernel
  ↓
CUDA-Q / GPU simulator
  ↓
Expectation value
  ↓
Classical optimizer 更新 parameters
  ↓
重複執行
  ↓
Sampling
  ↓
Max-Cut solution
```

目前先把 CUDA-Q 理解成一個可以整合 classical optimization 與 quantum kernel 的 programming model。

這次的重點是實際跑過一次 classical optimizer、quantum execution 到最後 sampling 的完整流程，並確認 CUDA-Q 可以使用 GPU simulator 執行。QAOA 本身的數學細節目前先不深入。


---

### 3-2. XACC & QCOR — Build from Source

> **Goal:** 在 Ubuntu 20.04 上從 source build XACC 與 QCOR，並使用 `qpp` simulator 執行簡單的 quantum kernel。

---

#### Environment

這部分按照指定環境使用 WSL2 Ubuntu 20.04：

```bash
lsb_release -a
gcc --version
g++ --version
python3 --version
```

主要環境：

- Ubuntu 20.04
- GCC / G++ 9.4
- Python 3.8
- Ninja 1.10
- LLVM 12 (`aideqc-llvm`)

---

#### Dependencies

先安裝基本的 build tools：

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    git \
    ninja-build \
    python3-dev \
    python3-pip
```

另外依照 AIDE-QC 提供的 repository 安裝 LLVM：

```bash
sudo apt-get install -y aideqc-llvm
```

LLVM 安裝位置：

```text
/usr/local/aideqc/llvm
```

---

#### Build XACC

取得 XACC source：

```bash
cd ~
git clone --recursive https://github.com/eclipse/xacc.git
cd xacc
mkdir build
cd build
```

一開始使用較新的 CMake 時遇到：

```text
Compatibility with CMake < 3.5 has been removed from CMake.
```

因此改用 Ubuntu 20.04 repository 提供的 CMake 3.16：

```bash
sudo apt install cmake
/usr/bin/cmake --version
```

重新建立乾淨的 `build` directory 後，使用：

```bash
/usr/bin/cmake .. -G Ninja
```

Configure 過程中曾在 sanitizer test 停留很久：

```text
Performing Test ADDRESS_SANITIZER_AVAILABLE
```

因此另外測試 AddressSanitizer：

```bash
g++ -fsanitize=address /tmp/asan_test.cpp -o /tmp/asan_test
/tmp/asan_test
```

確認 AddressSanitizer 本身可以正常執行。

改用 `/usr/bin/cmake` 並重新 configure 後成功完成：

```text
-- Configuring done
-- Generating done
-- Build files have been written to: /home/hoting/xacc/build
```

過程中有一些 optional packages 沒有安裝，例如：

```text
Qiskit not found
QSearch not found
Z3 library not found
```

目前先不處理，因為不影響這次基本的 XACC / QCOR 測試。

接著確認 CPU / memory：

```bash
nproc
free -h
```

這台電腦有 12 threads、16 GB RAM，因此使用：

```bash
ninja -j8
```

Build 最後完成：

```text
[807/807] Linking CXX shared library quantum/python/libxacc-quantum-py.so
```

安裝：

```bash
ninja install
```

XACC 被安裝到：

```text
~/.xacc
```

其中包含：

```text
~/.xacc/
├── bin/
├── include/
├── lib/
├── plugins/
└── py-plugins/
```

---

#### Build QCOR

接著取得 AIDE-QC 的 QCOR source：

```bash
cd ~
git clone --recursive https://github.com/aide-qc/qcor.git
cd qcor
mkdir build
cd build
```

Configure：

```bash
cmake .. -G Ninja
```

成功後：

```text
-- Configuring done
-- Generating done
-- Build files have been written to: /home/hoting/qcor/build
```

Build：

```bash
ninja -j8
```

完成：

```text
[231/231] Linking CXX shared library python/_pyqcor.so
```

安裝：

```bash
ninja install
```

QCOR 同樣安裝到：

```text
~/.xacc
```

接著設定 PATH 和 library path：

```bash
echo 'export PATH="$HOME/.xacc/bin:$PATH"' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH="$HOME/.xacc/lib:${LD_LIBRARY_PATH:-}"' >> ~/.bashrc
source ~/.bashrc
```

確認：

```bash
which qcor
```

結果：

```text
/home/hoting/.xacc/bin/qcor
```

---

#### QCOR Example

按照指定的測試方式，直接透過 stdin 將 C++ source 傳給 qcor：

```bash
printf "__qpu__ void f(qreg q) {
  H(q[0]);
  Measure(q[0]);
}
int main() {
  auto q = qalloc(1);
  f(q);
  q.print();
} #" | qcor -qpu qpp -shots 1024 -c c++ -

```

成功產生：

```text
a.out
```

執行：

```bash
./a.out
```

其中一次結果：

```text
"Measurements": {
    "0": 485,
    "1": 539
}
```

![Screenshot 2026-09-21 at 4.50.26 PM](https://hackmd.io/_uploads/ryO3O_RtGl.png)

總共：

```text
485 + 539 = 1024 shots
```

程式先對 qubit 執行 Hadamard gate，再進行 measurement，因此 `0` 和 `1` 的結果接近各一半，符合預期。

---

#### Problems Encountered

| Problem | Cause | Solution |
| --- | --- | --- |
| 新版 CMake configure XACC 失敗 | 舊版 XACC CMake 設定與新版 CMake policy 不相容 | 改用 Ubuntu 20.04 的 CMake 3.16 |
| Configure 在 sanitizer test 停留很久 | CMake 執行 dependency 的 sanitizer detection | 確認 sanitizer 可正常執行後，使用乾淨的 build directory 重新 configure |
| 部分 optional packages 找不到 | Qiskit、QSearch、Z3 等額外 dependency 未安裝 | 本次測試不需要，因此暫時不處理 |
| 指定的 stdin QCOR command 無法辨識 source | 目前 build 的 QCOR CLI 沒有正確辨識 stdin `-` | 改成建立 `test.cpp` 再交給 QCOR 編譯 |

---

#### Result

- [x] Ubuntu 20.04 build environment
- [x] XACC build from source
- [x] XACC installation
- [x] QCOR build from source
- [x] QCOR installation
- [x] 使用 QCOR 編譯 `__qpu__` C++ kernel
- [x] 使用 `qpp` simulator 執行
- [x] 成功取得 1024-shot measurement result

---

#### Current Understanding

這次實際跑過的流程可以簡化成：

```text
C++ source
    ↓
QCOR
    ↓
XACC
    ↓
qpp simulator
    ↓
measurement result
```

目前先把 QCOR 理解成 XACC 之上的 C++ frontend，讓 quantum kernel 可以直接寫在 C++ 程式中；XACC 則提供底下的 IR、compiler 與 accelerator framework。

這次主要先確認從 `__qpu__` kernel 到 simulator execution 的流程可以實際 build 並執行，還沒有深入追蹤 compiler 內部每個 transformation / pass。

實際操作後，這部分花比較多時間的地方不是 quantum circuit 本身，而是 XACC / QCOR 的 build environment、CMake 版本、dependencies 與 CLI 相容性問題。

---

## 4. Questions & Reflections

### Open Questions

**讀完 XACC 後：**

1. 如果想從頭實作一個類似 XACC 的 quantum-classical software stack，應該從哪一層開始？對碩士研究而言，做到什麼程度會是合理的 scope？
2. 如果基於 XACC、qcor 或其他 open-source framework 延伸，哪些 component 或 interface 適合作為進一步實作與研究的切入點？
3. XACC 用 IR 與 Accelerator interface 隔離不同 frontend / backend。這種 abstraction 在目前的 quantum-classical system 中是否仍然足夠？如果加入新的 execution model 或 backend，哪些地方最容易需要修改？

**讀完 qcor 後：**

1. 如果現在重新設計類似 qcor 的 compiler，還會採用相同架構嗎？還是會選擇 LLVM / MLIR / QIR 等較新的 compiler infrastructure？
2. qcor 已經提供 single-source C++ programming model，但如果未來要做到更 tightly-coupled CPU–QPU execution，現有的 compiler → runtime → backend interface 是否仍然足夠？

**完成 CUDA-Q / XACC / qcor 實作後：**

1. CUDA-Q 與 XACC / qcor 都在處理 heterogeneous quantum-classical programming，但兩者的 abstraction、IR、runtime 與 backend interface 有什麼主要差異？這些差異是因為年代不同，還是代表不同的 system design？
2. 目前使用 simulator 時，quantum execution 看起來只是一次 function / API call。但如果換成實際 QPU，compiler → runtime → backend 之間還會多出哪些 communication、scheduling 或 data movement 的問題？
3. QAOA 需要 classical optimizer 與 quantum kernel 反覆互動。如果把一次完整 QAOA execution 拆開量測，時間主要花在哪些部分？哪些 overhead 是 compiler 能改善的，哪些屬於 runtime / communication 的問題？
4. 這次 build XACC / qcor 時遇到不少 dependency、compiler version 與 interface compatibility 問題。對新的 quantum software stack 而言，如何設計比較穩定的 interface，讓 frontend、runtime 與 backend 可以獨立演進？
5. 如果要從目前的「成功執行 framework」進一步走向 research，下一步應該如何建立可以量測的 baseline？例如 compilation time、runtime overhead、CPU–QPU interaction latency 或不同 backend 的 execution behavior。

---

### Current Understanding

- XACC 將 QPU 抽象成 heterogeneous computing 中的 accelerator / co-processor，並透過 IR、transformation 與 Accelerator interface 隔離不同 quantum backend 的實作細節。
- qcor 建立在 XACC 之上，進一步把 quantum programming 整合進 C++，形成一條較完整的：
  **C++ source → compiler frontend → XACC → backend → execution** 路徑。
- 實際 build XACC / qcor 後，對 paper 中的 frontend、IR、runtime、backend 不再只是架構圖上的 component，而是開始能對應到實際的 source、library、compiler 與 simulator。
- CUDA-Q 的 Max-Cut 實作則讓我第一次實際跑過 hybrid workflow：
  **classical optimizer → quantum kernel → expectation value → parameter update → sampling**。
- 因此開始覺得 hybrid quantum-classical system 的問題不一定只存在於 quantum kernel 本身。當 classical 與 quantum execution 需要反覆互動時，compiler、runtime、communication 與 backend 都可能影響整體 execution。
- XACC / qcor 與 CUDA-Q 都提供 heterogeneous quantum-classical programming model，但實際架構與 abstraction 並不完全相同。比較不同世代 framework 如何處理相同問題，可能有助於理解哪些 design 已經被取代、哪些問題仍然存在。
- 目前也開始區分「工程問題」與「研究問題」。例如 dependency 或 CMake 相容性本身比較偏工程問題；如果某個 interface design 會造成可重現的 performance、portability 或 extensibility limitation，才比較可能進一步形成 research problem。

---

### Possible Directions

目前先不限定具體題目，而是把可能的切入點分成：

- **Compiler / IR**
  - quantum program representation
  - lowering / transformation
  - compiler optimization
  - 不同 IR / framework 之間的轉換

- **Runtime**
  - CPU–QPU execution coordination
  - hybrid workload scheduling
  - repeated quantum-classical execution
  - runtime overhead

- **Middleware / Interface**
  - compiler → runtime interface
  - runtime → backend interface
  - backend abstraction
  - communication / data movement

- **Backend Integration**
  - 加入或連接新的 simulator / QPU backend
  - 比較不同 backend 的 execution path
  - 觀察現有 abstraction 是否足夠

- **Performance / Profiling**
  - 拆解 end-to-end hybrid workflow
  - compilation time
  - runtime overhead
  - CPU–QPU interaction latency
  - 找出可能的 system bottleneck

> **目前想法：**  
> 現階段比起直接決定題目或重新實作完整的 XACC，我更想先沿著 **compiler → runtime → backend** 這條路徑繼續往下理解。
>
> 完成 CUDA-Q 與 XACC / qcor 的基本實作後，下一步除了「看懂架構」，也希望開始學習怎麼**觀察與量測一個 hybrid quantum-classical workflow**。如果能把 execution 拆成不同階段，再比較不同 framework / backend 的處理方式，應該會比較容易從單純的 framework 使用，慢慢找到可以被量測、比較與改善的 system-level problem。