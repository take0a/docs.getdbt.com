---
title: "HARファイルの生成方法"
description: "デバッグ用のHARファイルを生成する方法"
sidebar_label: 'Generate HAR files'
sidebar_position: 1
keywords:
  - HAR files
  - HTTP Archive
  - Troubleshooting
  - Debugging
---

HTTP アーカイブ (HAR) ファイルは、ユーザーのブラウザからデータを収集するために使用されます。dbt サポートは、このデータを使用してネットワークやリソースの問題をトラブルシューティングします。この情報には、ブラウザとサーバー間で行われたリクエストの詳細なタイミング情報が含まれます。

以下のセクションでは、[Google Chrome](#google-chrome)、[Mozilla Firefox](#mozilla-firefox)、[Apple Safari](#apple-safari)、[Microsoft Edge](#microsoft-edge) などの一般的なブラウザを使用して HAR ファイルを生成する方法について説明します。

:::info
HARファイルをdbt Labsに送信する前に、機密情報や個人を特定できる情報を削除または非表示にしてください。ファイルはテキストエディタで編集できます。
:::

### Google Chrome

1. Google Chrome を開きます。
2. **View** --> **Developer Tools** をクリックします。
3. **Network** タブを選択します。
4. Google Chrome が記録中であることを確認します。赤いボタン (🔴) は、すでに記録が進行中であることを示します。そうでない場合は、**Record network log** をクリックします。
5. **Preserve Log** を選択します。
6. **Clear network log** (🚫) をクリックして、既存のログを消去します。
7. 問題が発生したページに移動し、問題を再現します。
8. **Export HAR** (下矢印アイコン) をクリックして、ファイルを HAR としてエクスポートします。このアイコンは、**Clear network log** ボタンと同じ行にあります。
9. HAR ファイルを保存します。
10. HAR ファイルを dbt サポート チケット スレッドにアップロードします。

### Mozilla Firefox

1. Firefox を開きます。
2. アプリケーションメニューをクリックし、**More tools** --> **Web Developer Tools** を選択します。
3. 開発者ツールのドッキングタブで、**Network** を選択します。
4. 問題が発生したページに移動し、問題を再現します。ページを移動すると、自動的に記録が開始されます。
5. 完了したら、**Pause/Resume recording network log** をクリックします。
6. **File** 列の任意の場所を右クリックし、**Save All as HAR** を選択します。
7. HAR ファイルを保存します。
8. HAR ファイルを dbt サポートチケットスレッドにアップロードします。

### Apple Safari

1. Safari を開きます。
2. メニューバーに **Develop** メニューが表示されない場合は、**Safari** に移動して **Settings** を選択します。
3. **Advanced** をクリックします。
4. **Show features for web developers** チェックボックスをオンにします。
5. **Develop** メニューから **Show Web Inspector** を選択します。
6. **Network tab** をクリックします。
7. 問題が発生したページに移動し、問題を再現します。
8. 完了したら、**Export** をクリックします。
9. ファイルを保存します。
10. HAR ファイルを dbt サポートチケットスレッドにアップロードします。

### Microsoft Edge

1. Microsoft Edge を開きます。
2. ツールバーの右側にある **Settings and more** メニュー (...) をクリックし、**More tools** --> **Developer tools** を選択します。
3. **Network** をクリックします。
4. Microsoft Edge が記録中であることを確認します。赤いボタン (🔴) は、すでに記録が進行中であることを示します。そうでない場合は、**Record network log** をクリックします。
5. 問題が発生したページに移動し、問題を再現します。
6. 完了したら、**Stop recording network log** をクリックします。
7. **Export HAR** (下矢印アイコン) をクリックするか、**Ctrl + S** を押してファイルを HAR としてエクスポートします。
8. HAR ファイルを保存します。
9. HAR ファイルを dbt サポート チケット スレッドにアップロードします。

### 追加リソース
ChromeでHARファイルを生成する方法を視覚的に説明した[ChromeでHARファイルを生成する方法](https://www.loom.com/share/cabdb7be338243f188eb619b4d1d79ca)動画をご覧ください。