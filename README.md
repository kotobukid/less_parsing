# less parsing

JavascriptでSVG要素上に画像を作成し、ラスター画像として保存させたい。このときSVGをstyle要素で修飾している場合、DataURL経由でレンダーすると修飾情報が適用されない問題への対策を考える

```bash
npm run dev
```

# 既知の問題

Node用ライブラリのPostCSS(lessも？)を使用しているため、

* dev時には警告が出る
* build後は実行不能となる