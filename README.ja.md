# Kicad自動化ツール

## このパッケージには何が含まれているか

このプロジェクトは、[KiCad](https://kicad-pcb.org)向けの生産性向上ツールを多数パッケージ化しています。主に、Gitで追跡されるプロジェクトの製造出力とコマンドライン生産性の自動化に焦点を当てています。

### 製造出力

`make fabric-outputs`は、出力ディレクトリに次の出力を自動的に作成します:

* PDFおよびSVG形式の回路図
* .csvファイルとしてのBOM
*インタラクティブHTML BOMファイル
* DXF、ガーバー（ドリルファイルを含む）、PDF、およびSVGのレイアウトファイル。

これらはすべて他のプロジェクトに基づいています:

* [InteractiveHtmlBom](https://github.com/openscopeproject/InteractiveHtmlBom)
* [kicad-automation-script](https://github.com/productize/kicad-automation-scripts)
* [splitflap](https://github.com/scottbez1/splitflap)
* [kiplot](https://github.com/johnbeard/kiplot)


## Installation

### Prerequisites

この記事の執筆時点では、これらのツールはDockerコンテナで実行されているKiCad 5.1を対象としています。Dockerはわずかなオーバーヘッドを追加しますが、これらのツールを調整する複雑さを区分化するのに役立ち、（最も重要なポイントとして）ヘッドレスの仮想Xサーバを備えたUbuntuマシンの既知の構成でKiCadを実行することにより、「eeschema」から回路図出力を生成することができます。

理論的には、DockerizationはこれらのツールをWindowsまたはMacOSで実行することを可能にしますが、まだテストされていません。

このパッケージをセットアップする前に、ワークステーションにDockerとgitの両方をインストールする必要があります。

1. このパッケージをシステムのどこかにダウンロードしてインストールします。これから述べる手順のために、`/opt/kicad-tools/`に配置したと仮定します。
2. ローカルdockerコンテナをビルドしてデプロイします

    ```sh
    cd /opt/kicad-tools
    git submodule update --init
    make
    ```

    これはしばらくスピンし、Ubuntu、KiCad、および使用するさまざまなツールをダウンロードします。

    Dockerに接続できないというエラーが表示された場合、Dockerの構成により、Dockerを実行するために 'sudo'が必要になる場合があります。その場合、次のコマンドを実行してください:

    ```sh
    sudo make
    ```
    
    最終的に、'Successfully built 6476202f4575' のようなメッセージが出力されるはずです。

3. 'bin'ディレクトリをシェルのパスに追加します。通常、`.bashrc`かそれに準じる設定ファイルに以下の行を追加します:

    ```sh
    export PATH=/opt/kicad-tools/bin:$PATH
    ```
    
    先の手順で`sudo make`を実行する必要があった場合は、`PATH`の行のすぐ下に次の行も追加する必要があります:
    
    ```sh
    export DOCKER_NEEDS_SUDO=1
    ```
    
    シェルに変更を適用するために、シェルを再起動する必要があります。
    
    `sudo`なしでDockerコンテナを実行できるようにしたい場合は、こちらの手順を参照してください:
    https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user
    
    次のように入力することで、これまでに行ったすべてが機能していることを確認できます:
    
    ```sh
    kicad-docker-run hostname
    ```
    
    このようなものが出力されるはずです:
    
    ```
    $ kicad-docker-run hostname
    0230e9b27ebb
    ```
    
    エラーが発生しない場合は、問題はありません。

## プロジェクトの構成

### Makefile

サンプルのMakefileは、このパッケージとともに配布されます。これは`etc`ディレクトリにあります。

```
cp /opt/git-tools/etc/Makefile ./Makefile
```

**警告: Makefileが既にある場合は、このコマンドを盲目的に実行しないでください！**

### Git

To enable automatic graphical diffs of PCB layouts, you need to teach git how to handle
.kicad_pcb files

```
# echo "*.kicad_pcb diff=kicad_pcb" >> `git rev-parse --show-toplevel`/.gitattributes
# git config diff.kicad_pcb.command /opt/kicad-tools/bin/git-pcbdiff
```


## キャッシング

回路図とボードのdiff-able PNGの生成は、かなり遅い操作です。そのため、生成されたファイルをキャッシュします。デフォルトでは、それらは`/tmp`にキャッシュされます。各ボードと回路図はキャッシュ内で一意に識別されるため、複数のプロジェクト間でキャッシュを安全に共有できます。

再起動後も保持されるディレクトリをキャッシュすることを検討してください。

PCBイメージキャッシュを設定するには、環境変数 `BOARD_CACHE_DIR` を設定します。たとえば、以下の行を`.bashrc`に追加します:

```
export BOARD_CACHE_DIR=$HOME/.kicad-tools/cache/boards
```

回路図イメージキャッシュを設定するには、環境変数 `SCHEMATIC_CACHE_DIR` を設定します。たとえば、以下の行を`.bashrc`に追加します:

```
export SCHEMATIC_CACHE_DIR=$HOME/.kicad-tools/cache/schematics
```



## 使い方

### ビルドアーティファクトの生成

デフォルトでは、すべてのビルドアーティファクトはプロジェクトの'out'ディレクトリのサブディレクトリに作成されます。
アーティファクトが作成されるサブディレクトリの名前をカスタマイズするには、`BOARD_SNAPSHOT_LABEL` 環境変数を設定します。

出力ディレクトリをカスタマイズするには、OUTPUT_PATH環境変数を設定します。

製造出力のパッケージ全体を生成するには、

```
$ make fabrication-outputs
```

回路図のSVG画像を生成するには、
```
$ make schematic-svg
```

回路図のPDFを生成するには、
```
$ make schematic-pdf
```

基板レイアウトのガーバー、PDF、DXF、およびSVGを生成するには、
``` 
$ make gerbers
```

SVG形式のBOMを製造するには、
```
$ make bom
```

HTML形式のインタラクティブなBOMを生成するには、
```
$ make interactive-bom
```

何かをデバッグするためにdockerインスタンスにログインする必要がある場合、そのためのmakefileターゲットもあります:

```
$ make docker-shell
```

### Visual "diffs" between versions of a .kicad_pcb file

If you've configured git as described above, the regular `git diff` tool will show you a visual diff between two versions of a .kicad_pcb file.

In the future, this functionality may move to only running if you use `git difftool -gui`

To show the difference between your current checkout and the `HEAD` of your current branch, run

```
$ git diff HEAD my_board.kicad_pcb
```


### Visual "diffs" between versions of a .sch file

Due to some limitations in the information `git` provides to external git diff tools, you need to use a special command to compare schematics.

To compare the current checkout to the `HEAD` of your current branch

```
$ git schematic-diff HEAD my_board.sch
```

To compare the version of my_board.sch in `master` to the version as of tag `rev1`

```
$ git schematic-diff master rev1 my_board.sch
```
