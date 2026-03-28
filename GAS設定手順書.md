# Google Apps Script 設定手順

## ステップ1：スプレッドシートを作成

1. Google ドライブを開く（drive.google.com）
2. 「新規」→「Googleスプレッドシート」をクリック
3. 名前を「カヌースラローム集計」にする

## ステップ2：Apps Scriptを開く

1. スプレッドシートのメニューから「拡張機能」→「Apps Script」をクリック
2. 「コード.gs」というファイルが開く

## ステップ3：コードを貼り付ける

コード.gs の中身を全部消して、以下を貼り付けてください：

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    
    // シート名で新しいシートを作成（すでにある場合は削除して再作成）
    let sheet = ss.getSheetByName(data.sheetName);
    if (sheet) ss.deleteSheet(sheet);
    sheet = ss.insertSheet(data.sheetName);
    
    // ヘッダー行
    const gNums = Array.from({length: data.gateCount}, (_, i) => i + 1);
    const headers = ['発艇順','名前','カテゴリ','ラン','スタート','ゴール','差分',
      ...gNums.map(g => 'G' + g),
      'ペナルティ合計','タイム','ポイント'
    ];
    sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
    sheet.getRange(1, 1, 1, headers.length)
      .setBackground('#0f4c75').setFontColor('white').setFontWeight('bold');
    
    // データ行
    if (data.rows && data.rows.length > 0) {
      const rows = data.rows.map(r => [
        r['発艇順'], r['名前'], r['カテゴリ'], r['ラン'],
        r['スタート'], r['ゴール'], r['差分'],
        ...gNums.map(g => r['G' + g] || 0),
        r['ペナルティ合計'], r['タイム'], r['ポイント']
      ]);
      sheet.getRange(2, 1, rows.length, headers.length).setValues(rows);
    }
    
    // 列幅を自動調整
    sheet.autoResizeColumns(1, headers.length);
    
    return ContentService
      .createTextOutput(JSON.stringify({status: 'ok'}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({status: 'error', message: err.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  if (e.parameter.action === 'test') {
    return ContentService
      .createTextOutput(JSON.stringify({status: 'ok', message: '接続成功'}))
      .setMimeType(ContentService.MimeType.JSON);
  }
  return ContentService
    .createTextOutput(JSON.stringify({status: 'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## ステップ4：デプロイ（公開）

1. 右上の「デプロイ」→「新しいデプロイ」をクリック
2. 「種類の選択」で「ウェブアプリ」を選択
3. 「次のユーザーとして実行」→「自分」を選択
4. 「アクセスできるユーザー」→「全員」を選択
5. 「デプロイ」をクリック
6. Googleアカウントの確認画面が出たら「許可」をクリック
7. 「ウェブアプリのURL」をコピーする

## ステップ5：アプリに設定

1. カヌースラロームアプリの「大会設定」タブを開く
2. 「GAS URL」の欄にコピーしたURLを貼り付ける
3. 「接続テスト」ボタンを押して「接続成功！」と出ればOK

## 使い方

大会終了後、「スプレッドシートへ保存」ボタンを押してシート名を入力するだけで
Googleスプレッドシートに自動的にデータが保存されます。
