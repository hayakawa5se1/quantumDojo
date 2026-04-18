# quantumDojo6_1 で使われている PySCF 関連機能の一覧

対象ファイル: [`notebook/quantumDojo6_1.ipynb`](c:/Users/takat/Documents/program/qiskit/notebook/quantumDojo6_1.ipynb)

## 概要

このノートブックでは、PySCF を直接細かく呼び出す形ではなく、主に `openfermionpyscf.run_pyscf` を通して PySCF の機能を利用している。

そのため、コード上に明示的に書かれている関数は少ないが、内部では PySCF の分子軌道計算機能や電子状態計算機能が使われている。

以下では、ノートブック内で実際に利用されている PySCF 関連機能を、コード上の使い方と役割に分けて整理する。

## 一覧表

| 機能名 | コード上での使い方 | 役割 | 出力として使われているもの |
|---|---|---|---|
| PySCF 実行ラッパー | `run_pyscf(molecule, run_scf=True, run_fci=True)` | OpenFermion の `MolecularData` を受け取り、PySCF を使って量子化学計算を実行する | `hf_energy`, `fci_energy`, 積分データ |
| SCF 計算 | `run_scf=True` | Hartree-Fock の自己無撞着場計算を行う | `molecule.hf_energy` |
| Full-CI 計算 | `run_fci=True` | 指定基底内での厳密対角化に近い電子状態計算を行う | `molecule.fci_energy` |
| 1 電子積分の生成 | `molecule.one_body_integrals` | 分子軌道基底での 1 電子項を取得する | 1 電子ハミルトニアン成分 |
| 2 電子積分の生成 | `molecule.two_body_integrals` | 分子軌道基底での電子間相互作用を取得する | 2 電子ハミルトニアン成分 |
| 分子ハミルトニアン生成用データの供給 | `molecule.get_molecular_hamiltonian()` | PySCF が計算した積分値を用いて OpenFermion 側の分子ハミルトニアンを構成できるようにする | `molecular_hamiltonian` |

## 各機能の解説

### 1. `run_pyscf`

ノートブック中で最も重要な PySCF 関連機能は、次の行で使われている `run_pyscf` である。

```python
molecule = run_pyscf(molecule, run_scf=True, run_fci=True)
```

これは `openfermionpyscf` が提供するラッパー関数であり、OpenFermion の `MolecularData` オブジェクトを入力として受け取って、内部で PySCF を動かす。

この関数により、PySCF の結果が `molecule` オブジェクトに書き戻される。

### 2. SCF 計算

`run_scf=True` によって、自己無撞着場計算が実行される。

これは通常、Hartree-Fock 計算を意味しており、多電子系を平均場近似で扱う基本的な量子化学計算である。

ノートブックでは、その結果を

```python
molecule.hf_energy
```

として参照している。

### 3. Full-CI 計算

`run_fci=True` によって、Full-CI 計算が実行される。

これは、指定した基底の中で電子相関を完全に取り入れた計算であり、小さい分子では事実上の厳密解として扱える。

ノートブックでは、その結果を

```python
molecule.fci_energy
```

として参照している。

### 4. 1 電子積分

PySCF によって計算された分子軌道基底での 1 電子積分は

```python
molecule.one_body_integrals
```

として取得される。

この値は、電子の運動エネルギーや電子と原子核の引力相互作用に対応する項を構成する。

### 5. 2 電子積分

電子間クーロン反発に対応する 2 電子積分は

```python
molecule.two_body_integrals
```

として取得される。

この配列は、第二量子化ハミルトニアンの 2 電子項を作るために必要である。

### 6. 分子ハミルトニアンへの反映

PySCF が計算した積分値は、OpenFermion の

```python
molecule.get_molecular_hamiltonian()
```

を通じて分子ハミルトニアンへ変換される。

この関数自体は OpenFermion の機能だが、内部で使われる値は PySCF の計算結果である。したがって、PySCF はハミルトニアン構築の土台となる数値データを提供している。

## このノートブックにおける PySCF の位置づけ

このノートブックでは、PySCF は「量子化学の数値計算エンジン」として使われている。

役割を分けると次のようになる。

- OpenFermion: 分子データ管理、フェルミオン演算子生成、量子ビット変換
- PySCF: 分子軌道計算、積分計算、Hartree-Fock、Full-CI

つまり、PySCF が化学計算を担当し、OpenFermion が量子計算向けの形式へ橋渡しをしている。

## 注意点

ノートブックでは `pyscf` を直接 import していないため、見た目には PySCF の関数が少なく見える。しかし実際には `run_pyscf` の内部で PySCF の主要機能が利用されている。

そのため、このノートブックにおける PySCF の利用を理解するには、「明示的に書かれているコード」だけでなく、「`run_pyscf` が内部で実行している処理」を意識することが重要である。

## まとめ

`quantumDojo6_1.ipynb` で使われている PySCF 関連機能の中心は `run_pyscf` であり、その内部で SCF 計算、Full-CI 計算、1 電子積分・2 電子積分の生成が行われている。

このノートブックでは、PySCF が量子化学計算を担当し、その結果を OpenFermion が第二量子化ハミルトニアンや量子ビット演算子へ変換する、という役割分担になっている。
