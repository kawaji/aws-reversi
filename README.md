# EKS vs ECS Fargate リバーシゲーム コンテキスト

## 1. ゲームの基本構造

```
/reversi
├── index.html        # メインのHTMLファイル
├── styles.css        # CSSスタイル
├── script.js         # ゲームロジック
├── icons/            # AWSサービスアイコン
└── assets/           # その他のアセット（背景画像など）
```

## 2. アイコンの選定

- **EKS側（白コマ）**: Amazon EKSのアイコン (`./icons/Arch_Containers/64/Arch_Amazon-Elastic-Kubernetes-Service_64.png`)
- **ECS Fargate側（黒コマ）**: AWS Fargateのアイコン (`./icons/Arch_Containers/64/Arch_AWS-Fargate_64.png`)

## 3. ゲームデザイン要素

1. **ゲームタイトル**:
   - 「あなたはどっちのコンテナが好き？」

2. **ボード設計**:
   - 8x8の伝統的なリバーシボード
   - 各セルは白または黒の円形コマを配置
   - コマの中央にAWSサービスアイコンを表示

3. **視覚的デザイン**:
   - 白コマ: 白い円形の背景にEKSアイコンを配置
   - 黒コマ: 黒い円形の背景にECS Fargateアイコンを配置
   - ボード: AWSコンソールのような青系の色調

4. **ゲームフロー**:
   - 初期配置: 中央に2x2で交互に配置（伝統的なリバーシと同様）
   - 交互に手番を行い、相手のコマを挟むとひっくり返す
   - ゲーム終了時に多くのコマを持つ側が勝利

## 4. 実装のポイント

1. **アイコン表示**:
```javascript
// コマの描画例
function drawPiece(ctx, x, y, isEKS) {
  // 円形の背景を描画（白または黒）
  ctx.fillStyle = isEKS ? "#FFFFFF" : "#000000";
  ctx.beginPath();
  ctx.arc(x, y, PIECE_RADIUS, 0, Math.PI * 2);
  ctx.fill();
  
  // アイコンを中央に配置
  const icon = new Image();
  icon.src = isEKS ? "./icons/Arch_Containers/64/Arch_Amazon-Elastic-Kubernetes-Service_64.png" : 
                     "./icons/Arch_Containers/64/Arch_AWS-Fargate_64.png";
  
  // アイコンが読み込まれたら描画
  icon.onload = function() {
    const iconSize = PIECE_RADIUS * 1.2; // アイコンサイズ調整
    ctx.drawImage(icon, x - iconSize/2, y - iconSize/2, iconSize, iconSize);
  };
}
```

2. **ゲームロジック**:
```javascript
// リバーシの基本ロジック
function makeMove(row, col, isEKSTurn) {
  if (!isValidMove(row, col, isEKSTurn)) return false;
  
  // コマを置く
  board[row][col] = isEKSTurn ? 'EKS' : 'Fargate';
  
  // 8方向をチェックして挟まれたコマをひっくり返す
  const directions = [
    [-1, -1], [-1, 0], [-1, 1],
    [0, -1],           [0, 1],
    [1, -1],  [1, 0],  [1, 1]
  ];
  
  for (const [dx, dy] of directions) {
    flipPieces(row, col, dx, dy, isEKSTurn);
  }
  
  return true;
}
```

3. **タイトル表示**:
```html
<div class="game-title">
  <h1>あなたはどっちのコンテナが好き？</h1>
  <div class="subtitle">EKS vs ECS Fargate リバーシ対決</div>
</div>
```

## 5. テーマ設定

- **背景ストーリー**: EKS（Kubernetes）とECS Fargate（AWS独自のコンテナ管理）の対決
- **視覚的テーマ**: AWSコンソールのデザイン言語を取り入れる
- **サウンド**: オプションで、コマを置く際やひっくり返す際の効果音

## 6. 追加機能（オプション）

- **難易度レベル**: AIの強さを調整できる機能
- **ゲーム履歴**: 手の履歴を記録・再生できる機能
- **ヒント表示**: 有効な手を表示するヒント機能
- **リアルタイム対戦**: WebSocketを使用したオンライン対戦機能
- **コンテナ知識クイズ**: ゲーム中にEKSとECS Fargateに関する豆知識やクイズを表示

## 7. 基本的なHTMLテンプレート

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>あなたはどっちのコンテナが好き？ - EKS vs ECS Fargate リバーシ</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <div class="container">
    <div class="game-title">
      <h1>あなたはどっちのコンテナが好き？</h1>
      <div class="subtitle">EKS vs ECS Fargate リバーシ対決</div>
    </div>
    
    <div class="game-info">
      <div class="player eks-player">
        <img src="./icons/Arch_Containers/64/Arch_Amazon-Elastic-Kubernetes-Service_64.png" alt="EKS">
        <span>EKS: <span id="eks-score">2</span></span>
      </div>
      <div class="turn-indicator" id="turn-indicator">EKSの番です</div>
      <div class="player fargate-player">
        <img src="./icons/Arch_Containers/64/Arch_AWS-Fargate_64.png" alt="Fargate">
        <span>Fargate: <span id="fargate-score">2</span></span>
      </div>
    </div>
    
    <canvas id="game-board" width="480" height="480"></canvas>
    
    <div class="controls">
      <button id="new-game-btn">新しいゲーム</button>
      <button id="hint-btn">ヒント</button>
    </div>
    
    <div class="container-info">
      <div class="info-card">
        <h3>Amazon EKS</h3>
        <p>Amazon Elastic Kubernetes Serviceは、独自のKubernetesコントロールプレーンをプロビジョニングする必要なく、AWSで簡単にKubernetesを実行できるマネージドサービスです。</p>
      </div>
      <div class="info-card">
        <h3>AWS Fargate</h3>
        <p>AWS Fargateは、コンテナ用のサーバーレスコンピューティングエンジンで、Amazon ECSとAmazon EKSの両方で動作します。サーバーのプロビジョニングや管理を行わずにコンテナを実行できます。</p>
      </div>
    </div>
  </div>
  
  <script src="script.js"></script>
</body>
</html>
```
