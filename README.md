<!-- STREAMING_CHUNK: Writing README header and overview... -->
# 3D立体可視化: $x^2 + y^2 \le 1, \, 0 \le z \le x^2$

3次元座標空間において、条件 $x^2 + y^2 \le 1$ かつ $0 \le z \le x^2$ で定義される立体の構造をインタラクティブに観察できるWebベースの3Dグラフィックツールです。

Three.jsを利用してレンダリングされており、ブラウザ上で立体を自由な角度から回転・拡大縮小・スライス表示することができます。

---

## 📐 数学的背景

本アプリで扱う立体は、以下の3つの境界によって囲まれています。

1. **底面 ($z = 0$)**: 
   $xy$ 平面上の単位円板 ($x^2 + y^2 \le 1$)
2. **上面 ($z = x^2$)**: 
   $y$ 軸方向に伸びる放物柱面 (Parabolic Cylinder)
3. **側面 ($x^2 + y^2 = 1$)**: 
   単位円柱の側壁面

### 体積の計算 ($V$)
極座標変換 ($x = r\cos\theta, \, y = r\sin\theta$) を用いて二重積分を行うことで体積を求めることができます。

$$
V = \iint_{x^2+y^2 \le 1} x^2 \, dxdy = \int_{0}^{2\pi} \int_{0}^{1} (r\cos\theta)^2 \cdot r \, dr d\theta
$$

$$
V = \left( \int_{0}^{2\pi} \cos^2\theta \, d\theta \right) \left( \int_{0}^{1} r^3 \, dr \right) = \pi \cdot \frac{1}{4} = \frac{\pi}{4} \approx 0.785398
$$

---

<!-- STREAMING_CHUNK: Writing features and controls... -->
## ✨ 主な機能

- **3D自由視点操作**:
  - **ドラッグ (左クリック)**: 立体の回転
  - **右ドラッグ**: パニング (平行移動)
  - **スクロールホイール**: 拡大 / 縮小
- **要素の表示 / 非表示切り替え**:
  - 立体本体
  - 補助ガイド: 放物柱 ($z = x^2$)
  - 補助ガイド: 円柱 ($x^2 + y^2 = 1$)
- **不透明度調整スライダー**:
  - 立体本体の透明度を $0.2 \sim 1.0$ の範囲で変更可能
- **断面切断 (Clipping Plane) 機能**:
  - $y$ 軸方向に沿って断面をスライス切断して内部構造を観察可能
  - 「アニメーション」ボタンで自動スライス表示
- **レスポンシブ対応**:
  - PC・スマートフォン（タッチ操作）のどちらでも利用可能

---

<!-- STREAMING_CHUNK: Writing setup and technology stack... -->
## 🛠️ 使用技術

- **HTML5 / JavaScript (ES6+)**
- **[Three.js (r128)](https://threejs.org/)**: 3Dグラフィックスレンダリング & OrbitControls
- **[Tailwind CSS](https://tailwindcss.com/)**: UIデザイン・スタイリング
- **[MathJax v3](https://www.mathjax.org/)**: 数式レンダリング
- **[FontAwesome](https://fontawesome.com/)**: アイコンフォント

---

## 🚀 使い方 / 実行方法

1. リポジトリに含まれる `立体可視化_x2_y2_1_z_x2.html` ファイルを任意のWebブラウザ（Chrome, Edge, Firefox, Safari等）で直接開きます。
2. 外部ライブラリ（Three.js, Tailwind CSS等）をCDN経由で読み込むため、**インターネット接続が必要**です。
3. WebGLに対応している環境であれば、ビルド処理やサーバー構築なしで即座に動作します。

---

## 📄 ライセンス

MIT License
