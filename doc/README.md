# 国土数値情報ダウンロードサイト > 位置参照情報

- [国土数値情報ダウンロードサイト > 位置参照情報](https://nlftp.mlit.go.jp/cgi-bin/isj/dls/_choose_method.cgi)

## 処理

1. 位置参照情報は `Shift-JIS` なので、 `UTF-8` に変換する
2. `$ nkf --guess 01_2022.csv`
    - ファイルの文字コードを調べる
3. `$ nkf -w --overwrite 01_2022.csv`
    - 変換先の文字コードをオプションで指定する
      - -s: Shift-JIS
      - -e: EUC-JP
      - -w: UTF-8
4. `$ find . -type f -name "*.csv" | xargs nkf -w --overwrite`
    - 再起的にcsvファイルをUTF-8に変換する
