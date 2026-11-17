# container-vm-lab

「VM ってなに？」と「コンテナって結局なに？」を、手を動かして理解するための実験リポジトリ。

- VM は **ハードウェアを偽って OS を騙す**
- コンテナは **OS の見え方を偽ってアプリを騙す**

この違いを、自分の目で確かめるのがゴール。

## 進め方

**VM の基本 → コンテナ（ミニ Docker まで） → VM の発展** の順に進める。

## 準備

```bash
# このリポジトリの直下で実行する
limactl start --name=lab ./lima/lab.yaml
limactl shell lab
```

VM の中から、このリポジトリは Mac と同じパスで見える（`lima/lab.yaml` の `mounts` を参照）。

## 構成

```
container-vm-lab/
├── README.md     # ロードマップと進み具合
├── lima/         # Lima の VM 設定
├── notes/        # 手順ごとの記録
├── scripts/      # 実験用のシェルスクリプト（自分で書く）
└── minidocker/   # Go 言語のミニ Docker（自分で書く）
```

## ロードマップ

### VM の基本（半日）→ 記録は [notes/01-vm-basics.md](notes/01-vm-basics.md)

- [ ] 1. Mac と VM で `uname -a`・`free -h`・`df -h` を比べる
- [ ] 2. VM の中で `dmesg`・`systemd-analyze blame` を見て、カーネルから起動していることを確かめる
- [ ] 3. VM の中で `lscpu`・`lsblk`・`ip a` を見る。Mac 側で `~/.lima/lab/` とアクティビティモニタを見る
- [ ] 4. `lima/lab.yaml` の `cpus`・`memory` を変えて、VM を作り直す

### コンテナ（1日）→ 記録は [notes/02-container.md](notes/02-container.md)

- [ ] 1. VM に Docker を入れ、`docker run` したあと VM 側で `ps aux` を見る
- [ ] 2. `unshare` で PID・UTS・mount・net の namespace を1つずつ試す
- [ ] 3. Alpine のファイル一式を取ってきて `chroot` する
- [ ] 4. cgroup v2 で `memory.max` を設定し、上限を超えさせて OOM を起こす
- [ ] 5. Go 言語でミニ Docker を書く（2〜4を1つのプログラムにまとめる）
- [ ] 6. （余裕があれば）`ip link` で veth と bridge を作り、namespace 同士で `ping` を通す

### VM の発展（1日〜）

- [ ] 5. QEMU で Alpine Linux を手動で起動し、ファームウェア → カーネル → init の流れを追う
- [ ] 6. KVM API でミニハイパーバイザーを書く（VM の中で動かすので、入れ子の仮想化を使う）

## 参考

- Liz Rice「Containers From Scratch」
- LWN「Using the KVM API」（x86 向けなので、ARM64 では書き換えが必要）
