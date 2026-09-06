# API連携ガイド（仕事日記アプリ向け）

## 1. APIとは

API（Application Programming Interface）は、あるプログラムが別のプログラム（多くはサーバー）の機能やデータを、決まったルールで呼び出すための「窓口」です。

Web開発で最も多いのは **REST API**（HTTP経由でJSONをやり取りするAPI）です。基本の流れは次の3ステップです。

1. クライアント（ブラウザ）が `fetch()` でサーバーの決まったURL（エンドポイント）にHTTPリクエストを送る
2. サーバーが処理して結果をJSONで返す
3. クライアントがJSONを受け取り、画面に反映する

## 2. このアプリの現状

`index.html` を見ると、このアプリは今 **APIを一切使っていません**。

- タスクや日記データは `localStorage.setItem(...)` / `localStorage.getItem(...)` でブラウザ内だけに保存（`saveTasks()`, `saveDiary()`, `getEntries()` あたり）
- サーバー通信のコードはゼロ
- そのため「別の端末から見る」「データをバックアップする」といったことができない

## 3. API連携の具体例（このコードへの当てはめ）

### 例：日記データをサーバーに保存する場合

`saveDiary()`（`index.html` の602行目付近）を、`localStorage` への保存に加えて、サーバーAPIへも送信する形に変更します。

```js
async function saveDiaryToServer(entry) {
  const res = await fetch('https://your-api.example.com/entries', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(entry)
  });
  if (!res.ok) throw new Error('保存に失敗しました');
  return res.json();
}
```

`saveDiary()` の最後、`localStorage.setItem('diary_entries', ...)` の後ろに呼び出しを追加するイメージです。

```js
localStorage.setItem('diary_entries', JSON.stringify(entries));
localStorage.removeItem('draft_' + TODAY);

saveDiaryToServer(entry).catch(function() {
  showToast('オフラインのため端末にのみ保存しました');
});
```

### 例：起動時にサーバーから履歴を取得する場合

`getEntries()`（639行目付近）をサーバー取得に置き換える、または併用する形にします。

```js
async function fetchEntriesFromServer() {
  const res = await fetch('https://your-api.example.com/entries');
  return res.ok ? res.json() : [];
}
```

## 4. 連携する上で決めるべきこと

1. **バックエンドをどう用意するか**
   - 自前で作る（Node.js/ExpressやFirebase、Supabaseなど）
   - 既存のBaaS（Firebase Firestore、Supabaseなど）を使えばAPIサーバー自体を書かずに済む
2. **認証**
   - 個人用アプリでも複数端末で同期するならユーザー識別（トークンやログイン）が必要
3. **オフライン対応**
   - このアプリはPWA（`sw.js`あり）なのでオフラインでも動く。API連携時は通信失敗時に`localStorage`へフォールバックする設計が必須
4. **CORS**
   - フロント（このHTML）と別ドメインのAPIを呼ぶ場合、API側でCORSヘッダーの許可設定が必要

## まとめ

- 今は完全にローカル完結（API未使用）
- 連携する最短ルートは「`localStorage`の読み書き箇所を`fetch()`によるAPI呼び出しに置き換える／併用する」こと
- まずは保存先バックエンド（Firebase/Supabase等）を決めるのが最初の一歩
