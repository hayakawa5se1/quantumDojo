# quantumDojo6_1 コード構成の解説

対象ファイル: [`notebook/quantumDojo6_1.ipynb`](c:/Users/takat/Documents/program/qiskit/notebook/quantumDojo6_1.ipynb)

## 概要

このノートブックは、水素分子 `H2` を例として、OpenFermion と PySCF を使いながら量子化学ハミルトニアンを生成し、それを量子ビット演算子へ写像する流れを段階的に確認する構成になっている。

元の教材内容を一つの大きなコードブロックにせず、依存確認、分子定義、量子化学計算、ハミルトニアン生成、量子ビット変換、エネルギー曲線計算というモジュールごとに分けているため、各段階の役割が追いやすい。

## 全体の流れ

このノートブックの処理はおおむね次の順番で進む。

1. 必要なライブラリが入っているか確認する
2. OpenFermion と PySCF に必要な関数を import する
3. `H2` 分子の幾何構造と基底関数系を定義する
4. `run_pyscf` を使って Hartree-Fock と Full-CI を計算する
5. 分子軌道基底での 1 電子積分と 2 電子積分を確認する
6. 第二量子化形式の分子ハミルトニアンを作る
7. Jordan-Wigner 変換と Bravyi-Kitaev 変換で量子ビット演算子へ写像する
8. 疎行列化して基底エネルギーを求め、FCI エネルギーと比較する
9. H-H 距離を変えながらエネルギー曲線を描き、最後に計算時間を表示する

## 各セル群の役割

### 0. Dependency check

最初のセル群では、`openfermion`、`openfermionpyscf`、`pyscf`、`matplotlib`、`numpy` が利用可能かを確認している。

この部分の役割は、ノートブックの途中で `ModuleNotFoundError` が出るのを避け、必要なモジュールが不足している場合に早い段階で気づけるようにすることにある。

### 1. Imports

import セルでは、OpenFermion と PySCF の主要機能をまとめて読み込んでいる。

主な import は以下の通りである。

- `MolecularData`
- `run_pyscf`
- `get_fermion_operator`
- `jordan_wigner`
- `bravyi_kitaev`
- `get_sparse_operator`
- `get_ground_state`

この段階で、分子情報の保持、量子化学計算、第二量子化ハミルトニアン生成、量子ビット写像、基底状態計算までの一連の部品がそろう。

### 2. Define the H2 molecule

このセルでは、水素分子の幾何構造や計算条件を定義している。

主な変数は次の通りである。

- `basis`: 使う基底関数系。ここでは `sto-3g`
- `multiplicity`: スピン多重度。ここでは `1`
- `charge`: 分子の全電荷。ここでは `0`
- `distance`: H-H 間距離
- `geometry`: 原子位置
- `description`: 計算条件を区別する文字列

その後、`MolecularData(...)` を使って分子情報オブジェクトを生成している。

### 3. Run PySCF

ここでは

```python
molecule = run_pyscf(molecule, run_scf=True, run_fci=True)
```

を実行している。

この処理により、PySCF を用いた分子軌道計算が実行され、`molecule` オブジェクトの中に Hartree-Fock エネルギー、Full-CI エネルギー、1 電子積分、2 電子積分などが保存される。

ノートブックの中で、量子化学的な中核計算を担っているのがこの部分である。

### 4. Hartree-Fock and Full-CI energies

このセルでは、`molecule.hf_energy` と `molecule.fci_energy` を表示している。

- `hf_energy`: 平均場近似に基づく Hartree-Fock エネルギー
- `fci_energy`: 与えられた基底の中での厳密解に近い Full-CI エネルギー

これにより、簡易近似と厳密計算の差を比較できる。

### 5. One-body and two-body integrals

ここでは

- `molecule.one_body_integrals`
- `molecule.two_body_integrals`

を表示している。

これらは第二量子化ハミルトニアンを構成する材料である。

- 1 電子積分は運動エネルギーや原子核との相互作用に関わる
- 2 電子積分は電子間クーロン相互作用に関わる

この段階で、量子化学計算の出力が数値としてどのように格納されているかを確認できる。

### 6. Second-quantized Hamiltonian

この部分では

```python
molecular_hamiltonian = molecule.get_molecular_hamiltonian()
fermion_hamiltonian = get_fermion_operator(molecular_hamiltonian)
```

を使って、分子ハミルトニアンを第二量子化形式に変換している。

`molecular_hamiltonian` は相互作用演算子の形で持たれ、`fermion_hamiltonian` は生成消滅演算子で書かれた `FermionOperator` になる。

ここで初めて、量子化学計算の結果が「量子アルゴリズムで使える演算子」として整理される。

### 7. Map to qubit operators

このセルでは、フェルミオン演算子を量子ビット演算子へ写像している。

```python
jw_hamiltonian = jordan_wigner(fermion_hamiltonian)
bk_hamiltonian = bravyi_kitaev(fermion_hamiltonian)
```

役割は次の通りである。

- `jordan_wigner`: Jordan-Wigner 変換
- `bravyi_kitaev`: Bravyi-Kitaev 変換

これにより、フェルミオン系の問題が Pauli 演算子の和として書き直され、量子回路ベースのアルゴリズムに入力できる形になる。

### 8. Sparse-matrix ground-state energy

ここでは量子ビットハミルトニアンを疎行列へ変換し、基底状態エネルギーを数値的に求めている。

```python
jw_sparse = get_sparse_operator(jw_hamiltonian)
ground_energy, ground_state = get_ground_state(jw_sparse)
```

その結果を `molecule.fci_energy` と比較することで、変換後のハミルトニアンが正しく元の分子問題を表しているか確認している。

### 9. Energy curve as a function of bond length

最後の部分では、H-H 距離を変えながらエネルギー曲線を計算している。

まず `compute_h2_energy(distance, ...)` という補助関数を定義し、与えられた距離ごとに `MolecularData` を作成し直して `run_pyscf` を実行する構造になっている。

その後、

```python
distances = np.linspace(0.4, 1.0, 7)
results = [compute_h2_energy(d) for d in distances]
```

で 0.4 Å から 1.0 Å までの範囲を走査している。

さらに

```python
start_time = time.perf_counter()
...
elapsed_time = time.perf_counter() - start_time
print(f"Elapsed time: {elapsed_time:.3f} s")
```

によって、距離スキャン全体にかかった計算時間も最後に表示している。

## このノートブックの構造上の特徴

このノートブックの良い点は、処理の責務が段階ごとに分離されていることである。

- 依存確認は依存確認だけ
- 分子定義は分子定義だけ
- 量子化学計算は `run_pyscf` に集中
- 変換処理は変換処理として独立
- 距離走査は補助関数で再利用可能

そのため、後から VQE のセルを追加したり、別の分子へ差し替えたりしやすい構成になっている。

## まとめ

`quantumDojo6_1.ipynb` は、OpenFermion と PySCF を連携させて分子ハミルトニアンを作り、量子ビット表現へ変換するまでを見通しよく学べるノートブックである。

特に、分子定義、PySCF 実行、第二量子化、量子ビット写像、距離依存のエネルギー計算を別々のモジュールとして分けているため、各処理の役割を理解しやすい。
