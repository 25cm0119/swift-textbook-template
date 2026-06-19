# 第6章：ジェスチャー操作

> 執筆者： アントニオ
> 最終更新：2026-06-17

## この章で学ぶこと

5つの主要なジェスチャー操作（タップ、ロングプレス、ドラッグ、ピンチ、回転）を実装する方法を学びます。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
import SwiftUI

// MARK: - メインビュー
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("タップ & ロングプレス") {
                    TapDemoView()
                }
                NavigationLink("ドラッグ") {
                    DragDemoView()
                }
                NavigationLink("ピンチ（拡大縮小）") {
                    MagnifyDemoView()
                }
                NavigationLink("回転") {
                    RotateDemoView()
                }
                NavigationLink("組み合わせ") {
                    CombinedDemoView()
                }
            }
            .navigationTitle("ジェスチャー体験")
        }
    }
}

// MARK: - タップ & ロングプレス
struct TapDemoView: View {
    @State private var tapCount = 0
    @State private var backgroundColor: Color = .blue
    @State private var isPressed = false

    var body: some View {
        VStack(spacing: 30) {
            Text("タップ回数: \(tapCount)")
                .font(.title)

            // シングルタップ
            RoundedRectangle(cornerRadius: 16)
                .fill(backgroundColor)
                .frame(width: 200, height: 200)
                .overlay {
                    Text("タップしてね")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .onTapGesture {
                    tapCount += 1
                    backgroundColor = Color(
                        hue: Double.random(in: 0...1),
                        saturation: 0.7,
                        brightness: 0.9
                    )
                }

            // ロングプレス
            Circle()
                .fill(isPressed ? .green : .orange)
                .frame(width: 120, height: 120)
                .scaleEffect(isPressed ? 1.3 : 1.0)
                .overlay {
                    Text(isPressed ? "成功!" : "長押し")
                        .foregroundStyle(.white)
                        .font(.headline)
                }
                .animation(.spring(duration: 0.3), value: isPressed)
                .onLongPressGesture(minimumDuration: 1.0) {
                    isPressed = true
                    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                        isPressed = false
                    }
                }
        }
        .navigationTitle("タップ & ロングプレス")
    }
}

// MARK: - ドラッグ
struct DragDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero

    var body: some View {
        VStack {
            Text("カードをドラッグしてみよう")
                .font(.headline)
                .padding()

            Spacer()

            RoundedRectangle(cornerRadius: 20)
                .fill(
                    LinearGradient(
                        colors: [.purple, .blue],
                        startPoint: .topLeading,
                        endPoint: .bottomTrailing
                    )
                )
                .frame(width: 200, height: 280)
                .shadow(radius: 8)
                .overlay {
                    VStack {
                        Image(systemName: "hand.draw")
                            .font(.system(size: 40))
                        Text("ドラッグ")
                            .font(.title3)
                    }
                    .foregroundStyle(.white)
                }
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ドラッグ")
    }
}

// MARK: - ピンチ（拡大縮小）
struct MagnifyDemoView: View {
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0

    var body: some View {
        VStack {
            Text("ピンチで拡大縮小")
                .font(.headline)
                .padding()

            Text(String(format: "倍率: %.1fx", scale))
                .font(.caption)
                .foregroundStyle(.secondary)

            Spacer()

            Image(systemName: "star.fill")
                .font(.system(size: 100))
                .foregroundStyle(.yellow)
                // タッチ判定を300×300の透明な領域に広げる
                .frame(width: 300, height: 300)
                .contentShape(Rectangle())
                .scaleEffect(scale)
                .gesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    scale = 1.0
                    lastScale = 1.0
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("ピンチ")
    }
}

// MARK: - 回転
struct RotateDemoView: View {
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero

    var body: some View {
        VStack {
            Text("2本指で回転")
                .font(.headline)
                .padding()

            Text(String(format: "角度: %.0f°", angle.degrees))
                .font(.caption)
                .foregroundStyle(.secondary)

            Spacer()

            Image(systemName: "arrow.up")
                .font(.system(size: 80))
                .foregroundStyle(.red)
                // タッチ判定を300×300の透明な領域に広げる
                .frame(width: 300, height: 300)
                .contentShape(Rectangle())
                .rotationEffect(angle)
                .gesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("回転")
    }
}

// MARK: - 組み合わせ
struct CombinedDemoView: View {
    @State private var offset: CGSize = .zero
    @State private var lastOffset: CGSize = .zero
    @State private var scale: CGFloat = 1.0
    @State private var lastScale: CGFloat = 1.0
    @State private var angle: Angle = .zero
    @State private var lastAngle: Angle = .zero

    var body: some View {
        VStack {
            Text("ドラッグ・ピンチ・回転を同時に")
                .font(.headline)
                .padding()

            Spacer()

            Image(systemName: "photo.artframe")
                .font(.system(size: 120))
                .foregroundStyle(.indigo)
                // タッチ判定を300×300の透明な領域に広げる
                .frame(width: 300, height: 300)
                .contentShape(Rectangle())
                .scaleEffect(scale)
                .rotationEffect(angle)
                .offset(offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = CGSize(
                                width: lastOffset.width + value.translation.width,
                                height: lastOffset.height + value.translation.height
                            )
                        }
                        .onEnded { _ in
                            lastOffset = offset
                        }
                )
                .simultaneousGesture(
                    MagnifyGesture()
                        .onChanged { value in
                            scale = lastScale * value.magnification
                        }
                        .onEnded { _ in
                            lastScale = scale
                        }
                )
                .simultaneousGesture(
                    RotateGesture()
                        .onChanged { value in
                            angle = lastAngle + value.rotation
                        }
                        .onEnded { _ in
                            lastAngle = angle
                        }
                )

            Spacer()

            Button("リセット") {
                withAnimation(.spring) {
                    offset = .zero
                    lastOffset = .zero
                    scale = 1.0
                    lastScale = 1.0
                    angle = .zero
                    lastAngle = .zero
                }
            }
            .buttonStyle(.bordered)
            .padding()
        }
        .navigationTitle("組み合わせ")
    }
}

#Preview {
    ContentView()
}
```

**このアプリは何をするものか：**

タップでカウントや色変更、ロングプレスで状態変更、ドラッグでカード移動、ピンチで拡大縮小、回転で角度変更。組み合わせでは、3つのジェスチャーを同時に実行するできます。

## コードの詳細解説

### 基本ジェスチャー（タップ、ロングプレス）

```swift
// シングルタップ
RoundedRectangle(cornerRadius: 16)
    .fill(backgroundColor)
    .frame(width: 200, height: 200)
    .overlay {
        Text("タップしてね")
            .foregroundStyle(.white)
            .font(.headline)
    }
    .onTapGesture {
        tapCount += 1
        backgroundColor = Color(
            hue: Double.random(in: 0...1),
            saturation: 0.7,
            brightness: 0.9
        )
    }

// ロングプレス
Circle()
    .fill(isPressed ? .green : .orange)
    .frame(width: 120, height: 120)
    .scaleEffect(isPressed ? 1.3 : 1.0)
    .overlay {
        Text(isPressed ? "成功!" : "長押し")
            .foregroundStyle(.white)
            .font(.headline)
    }
    .animation(.spring(duration: 0.3), value: isPressed)
    .onLongPressGesture(minimumDuration: 1.0) {
        isPressed = true
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            isPressed = false
        }
    }
```

**何をしているか：**
タップジェスチャーはビューをタップしたときにonTapGestureのクロージャが実行されます。このコードでは、タップされるたびにtapCountを+1してカウントアップし、同時にランダムな色に背景色を変更しています。

ロングプレスジェスチャーは、指を最小1秒間押し続けたときに実行されます。押している間にisPressedがtrueになり、円の色が緑に変わり、サイズが1.3倍になります。

**なぜこう書くのか：**
タップとロングプレスは、タップジェスチャーのビルダーパターンで実装するのが最もシンプルです。onTapGestureとonLongPressGestureモディファイアは、複雑なDragGestureなどと違い、最小限のコードで実装できます。

**もしこう書かなかったら：**
- `onTapGesture`を使わずに手動でタップ判定をしようとすると、複雑な座標計算が必要になり、実装が非常に難しくなります。
- `minimumDuration`を設定しないと、短いタップでもロングプレスが反応してしまい、両者の区別ができなくなります。
- `animation`モディファイアを追加しないと、色とサイズが瞬間的に変わるだけで、視覚的なフィードバックが不足します。実際に試すと、ユーザーが反応したことが実感しにくくなります。

---

### ドラッグジェスチャーとオフセット管理

```swift
@State private var offset: CGSize = .zero
@State private var lastOffset: CGSize = .zero

RoundedRectangle(cornerRadius: 20)
    .fill(...)
    .frame(width: 200, height: 280)
    .offset(offset)
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = CGSize(
                    width: lastOffset.width + value.translation.width,
                    height: lastOffset.height + value.translation.height
                )
            }
            .onEnded { _ in
                lastOffset = offset
            }
    )
```

**何をしているか：**
ドラッグジェスチャーでカードを移動させます。`DragGesture().onChanged`は指がスクリーン上を動く間、毎フレーム実行されます。`value.translation`は現在の移動量（最初のタッチ位置との差分）を表すため、`lastOffset`に加算して累積オフセットを計算します。ドラッグが終了したら、`onEnded`で`lastOffset`を更新して、次のドラッグの開始点を記録します。

**なぜこう書くのか：**
`lastOffset`と`offset`の2つの状態変数が必要です。理由は、`DragGesture().onChanged`の`value.translation`は常に最後のドラッグ開始点からの相対移動だからです。新しいドラッグが開始されるたびに`translation`がリセットされるため、`lastOffset`に前回のドラッグ終了時点を保存しておかないと、カードが想定外の位置にジャンプしてしまいます。

**もしこう書かなかったら：**
- `lastOffset`を使わず、単に`offset = value.translation`と書くと、新しいドラッグを開始するたびにカードが初期位置にジャンプします。
- `.onEnded`で`lastOffset = offset`の更新を忘れると、ドラッグ中に複数のジェスチャーを検出してしまい、予測不可能な動きになります。
- 実際に試してみたら、`lastOffset`の更新なしでは、2回目のドラッグでカードが最初の位置に戻ってしまいました。


---

### 拡大縮小と回転

```swift
@State private var scale: CGFloat = 1.0
@State private var lastScale: CGFloat = 1.0

Image(systemName: "star.fill")
    .font(.system(size: 100))
    .foregroundStyle(.yellow)
    .frame(width: 300, height: 300)
    .contentShape(Rectangle())
    .scaleEffect(scale)
    .gesture(
        MagnifyGesture()
            .onChanged { value in
                scale = lastScale * value.magnification
            }
            .onEnded { _ in
                lastScale = scale
            }
    )

// 回転の例
@State private var angle: Angle = .zero
@State private var lastAngle: Angle = .zero

Image(systemName: "arrow.up")
    .font(.system(size: 80))
    .foregroundStyle(.red)
    .frame(width: 300, height: 300)
    .contentShape(Rectangle())
    .rotationEffect(angle)
    .gesture(
        RotateGesture()
            .onChanged { value in
                angle = lastAngle + value.rotation
            }
            .onEnded { _ in
                lastAngle = angle
            }
    )```

**何をしているか：**

**なぜこう書くのか：**

**もしこう書かなかったら：**

---

### ジェスチャーの組み合わせとアニメーション

```swift
// 該当部分のコードを抜粋して貼る
```

**何をしているか：**
ピンチジェスチャーで星のアイコンを拡大・縮小します。`MagnifyGesture`の`value.magnification`は、2本指の間隔の比率を示します。前フレームの倍率`lastScale`に`magnification`を掛けることで、段階的な拡大縮小を実現します。

回転ジェスチャーで矢印を回転させます。`RotateGesture`の`value.rotation`は、2本指がどれだけ回転したかを`Angle`で取得できます。前フレームの角度`lastAngle`に新しい回転角を足すことで、連続的な回転を表現します。

**なぜこう書くのか：**

ドラッグと同じく、ピンチと回転でも「前フレームからの変化量」を累積する必要があります。`value.magnification`は新しいジェスチャーが開始されるたびにリセットされるため、`lastScale`に保存して累積させないと、拡大中に指が画面から離れると元のサイズに戻ってしまいます。

`.contentShape(Rectangle())`を追加している理由は、画像の透明な部分を避けてタッチ判定をするためです。このテクニックを使わないと、アイコンの外側（フレームの300×300の領域内でも透明な部分）をタッチしてもジェスチャーが反応しません。

**もしこう書かなかったら：**

- `lastScale * value.magnification`ではなく、単に`value.magnification`と書くと、ピンチを開始するたびに元のサイズに戻ります。
- `.contentShape(Rectangle())`を使わないと、アイコンの細い部分（矢印の先端など）しかタッチできなくなり、ユーザー体験が悪くなります。実際に試すと、クリックできる領域が非常に小さく、操作が困難になりました。
- `Angle`型ではなく`CGFloat`型を使うと、角度の単位を自分で管理する必要があり、ラジアンと度数法の変換バグが生じやすくなります。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| `onTapGesture` | ビューのタップを検出するモディファイア | `.onTapGesture { tapCount += 1 }` |
| `onLongPressGesture` | 長押しを検出するモディファイア | `.onLongPressGesture(minimumDuration: 1.0) { ... }` |
| `DragGesture` | ドラッグの移動量を取得するジェスチャー | `.gesture(DragGesture().onChanged { value.translation })` |
| `MagnifyGesture` | ピンチジェスチャーで拡大・縮小を認識 | `.gesture(MagnifyGesture().onChanged { scale *= value.magnification })` |
| `RotateGesture` | 2本指の回転を認識するジェスチャー | `.gesture(RotateGesture().onChanged { angle += value.rotation })` |
| `scaleEffect` | ビューのサイズを倍率で変更するモディファイア | `.scaleEffect(1.5)` |
| `rotationEffect` | ビューを角度で回転させるモディファイア | `.rotationEffect(.degrees(45))` |
| `offset` | ビューを相対位置で移動させるモディファイア | `.offset(CGSize(width: 10, height: 20))` |
| `contentShape` | タッチ判定の有効範囲を定義するモディファイア | `.contentShape(Rectangle())` |
| `simultaneousGesture` | 複数のジェスチャーを同時に認識させるモディファイア | `.simultaneousGesture(MagnifyGesture())` |

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1：**
- やったこと：
- 結果：
- わかったこと：

**実験2：**
- やったこと：
- 結果：
- わかったこと：

## AIに聞いて特に理解が深まった質問 TOP3

1. **質問：**
   **得られた理解：**

2. **質問：**
   **得られた理解：**

3. **質問：**
   **得られた理解：**

## この章のまとめ

（この章で学んだ最も重要なことを、未来の自分が読み返したときに役立つように書く）
