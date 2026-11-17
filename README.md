# container-vm-lab

VM とコンテナを自分で作って、仕組みを理解するための実験リポジトリ。

最終的に、次の2つを自作する。

| 作るもの | 中身 | 置き場所 |
|---|---|---|
| ミニ Docker | namespace・cgroup・chroot を組み合わせて、隔離したプロセスを起動する Go のプログラム | `minidocker/` |
| ミニハイパーバイザー | KVM API で仮想 CPU とメモリを用意し、ゲストのコードを動かすプログラム | `minihypervisor/` |

作りながら、次の違いを自分の目で確かめる。

- VM はハードウェアを仮想化し、その上で OS を丸ごと動かす
- コンテナはホストのカーネルを共有したまま、プロセスを隔離して動かす

## 進め方

次の3つのステップで進める。コードとスクリプトは自分で書く。

1. VM を観察する：Lima で Ubuntu の VM を立て、VM の CPU・メモリ・ディスクが仮想化されたものであることを確かめる（半日）
2. コンテナを自作する：部品を1つずつ試してから、ミニ Docker に組み立てる（1日）
3. VM を自作する：QEMU で起動の流れを追ってから、ミニハイパーバイザーを書く（1日〜）

## 準備

```bash
# このリポジトリの直下で実行する
limactl start --name=lab ./lima/lab.yaml
limactl shell lab
```

VM の中から、このリポジトリは Mac と同じパスで見える（`lima/lab.yaml` の `mounts` を参照）。入れ子の仮想化（`nestedVirtualization`）を有効にしてあるので、ステップ3の KVM もこの VM の中で使える（Apple M3 以降が必要）。

## 構成

```
container-vm-lab/
├── README.md         # ロードマップと進み具合
├── lima/             # Lima の VM 設定
├── notes/            # 手順ごとの記録
├── scripts/          # 実験用のシェルスクリプト（自分で書く）
├── minidocker/       # ミニ Docker（自分で書く）
└── minihypervisor/   # ミニハイパーバイザー（自分で書く）
```

## ステップ1：VM を観察する（半日）

記録は [notes/01-vm-basics.md](notes/01-vm-basics.md) に書く。

- [ ] 1. Mac と VM で `uname -a`・`free -h`・`df -h` を比べる
- [ ] 2. VM の中で `dmesg`・`systemd-analyze blame` を見て、カーネルから起動していることを確かめる
- [ ] 3. VM の中で `lscpu`・`lsblk`・`ip a` を見る。Mac 側で `~/.lima/lab/` とアクティビティモニタを見る
- [ ] 4. `lima/lab.yaml` の `cpus`・`memory` を変えて、VM を作り直す

## ステップ2：コンテナを自作する（1日）

記録は [notes/02-container.md](notes/02-container.md) に書く。

部品を1つずつ試す。

- [ ] 1. VM に Docker を入れ、`docker run` したあと VM 側で `ps aux` を見る
- [ ] 2. `unshare` で PID・UTS・mount・net の namespace を1つずつ試す
- [ ] 3. Alpine のファイル一式を取ってきて `chroot` する
- [ ] 4. cgroup v2 で `memory.max` を設定し、上限を超えさせて OOM を起こす

部品を組み立てる。

- [ ] 5. Go 言語でミニ Docker を書く（2〜4を1つのプログラムにまとめる）
- [ ] 6. （余裕があれば）`ip link` で veth と bridge を作り、ミニ Docker のコンテナ同士で `ping` を通す

## ステップ3：VM を自作する（1日〜）

部品を試す。

- [ ] 1. QEMU で Alpine Linux を手動で起動し、ファームウェア → カーネル → init の流れを追う

部品を組み立てる。

- [ ] 2. KVM API でミニハイパーバイザーを書く（仮想 CPU とメモリを用意し、ゲストのコードを動かす）

## 参考

- Liz Rice「Containers From Scratch」
- LWN「Using the KVM API」（x86 向けなので、ARM64 では書き換えが必要）
