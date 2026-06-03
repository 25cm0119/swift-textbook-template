# 第2章：地図アプリの基本

> 執筆者：アントニオ
> 最終更新：2026/05/08

## この章で学ぶこと

MapKitを使ってアプリ内に地図を表示し、特定の位置にマーカーを配置する方法を学ぶ。具体的には東京の観光スポットをランドマークデータとして構造体で定義し、地図上にアイコン付きマーカーで表示して、カテゴリでフィルターするアプリを題材にする。ユーザーが地図を操作しながら、興味のあるカテゴリのスポットだけを表示・非表示にできる機能を実装する。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// ============================================
// 第2章（基本）：MapKitで地図を表示するアプリ
// ============================================
// 東京の観光スポットを地図上にマーカーで表示します。
// マーカーをタップすると詳細情報が表示されます。
// ============================================

import SwiftUI
import MapKit

// MARK: - データモデル

struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let description: String
    let coordinate: CLLocationCoordinate2D
    let category: Category

    enum Category: String, CaseIterable {
        case temple = "寺社"
        case tower = "タワー"
        case park = "公園"

        var iconName: String {
            switch self {
            case .temple: return "building.columns"
            case .tower: return "antenna.radiowaves.left.and.right"
            case .park: return "leaf"
            }
        }

        var color: Color {
            switch self {
            case .temple: return .red
            case .tower: return .blue
            case .park: return .green
            }
        }
    }
}

// MARK: - サンプルデータ

extension Landmark {
    static let sampleData: [Landmark] = [
        Landmark(
            name: "浅草寺",
            description: "東京都内最古の寺院。雷門が有名。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7148, longitude: 139.7967),
            category: .temple
        ),
        Landmark(
            name: "東京タワー",
            description: "1958年に完成した高さ333mの電波塔。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6586, longitude: 139.7454),
            category: .tower
        ),
        Landmark(
            name: "東京スカイツリー",
            description: "高さ634mの世界一高い自立式電波塔。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7101, longitude: 139.8107),
            category: .tower
        ),
        Landmark(
            name: "明治神宮",
            description: "明治天皇と昭憲皇太后を祀る神社。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6764, longitude: 139.6993),
            category: .temple
        ),
        Landmark(
            name: "上野恩賜公園",
            description: "美術館や動物園がある広大な公園。",
            coordinate: CLLocationCoordinate2D(latitude: 35.7146, longitude: 139.7732),
            category: .park
        ),
        Landmark(
            name: "新宿御苑",
            description: "都心にある広さ58.3ヘクタールの庭園。",
            coordinate: CLLocationCoordinate2D(latitude: 35.6852, longitude: 139.7100),
            category: .park
        ),
    ]
}

// MARK: - メインビュー

struct ContentView: View {
    @State private var cameraPosition: MapCameraPosition = .region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
            span: MKCoordinateSpan(latitudeDelta: 0.08, longitudeDelta: 0.08)
        )
    )
    @State private var selectedLandmark: Landmark?
    @State private var selectedCategories: Set<Landmark.Category> = Set(Landmark.Category.allCases)

    var filteredLandmarks: [Landmark] {
        Landmark.sampleData.filter { selectedCategories.contains($0.category) }
    }

    var body: some View {
        ZStack(alignment: .bottom) {
            // 地図
            Map(position: $cameraPosition) {
                ForEach(filteredLandmarks) { landmark in
                    Marker(
                        landmark.name,
                        systemImage: landmark.category.iconName,
                        coordinate: landmark.coordinate
                    )
                    .tint(landmark.category.color)
                }
            }
            .mapStyle(.standard(elevation: .realistic))

            // カテゴリフィルター
            VStack(spacing: 8) {
                if let landmark = selectedLandmark {
                    LandmarkCard(landmark: landmark)
                        .transition(.move(edge: .bottom))
                }

                CategoryFilter(selectedCategories: $selectedCategories)
            }
            .padding()
        }
        .onMapCameraChange { context in
            // 地図の操作に応じた処理を追加できる
        }
    }
}

// MARK: - カテゴリフィルター

struct CategoryFilter: View {
    @Binding var selectedCategories: Set<Landmark.Category>

    var body: some View {
        HStack(spacing: 8) {
            ForEach(Landmark.Category.allCases, id: \.self) { category in
                Button {
                    if selectedCategories.contains(category) {
                        selectedCategories.remove(category)
                    } else {
                        selectedCategories.insert(category)
                    }
                } label: {
                    HStack(spacing: 4) {
                        Image(systemName: category.iconName)
                        Text(category.rawValue)
                    }
                    .font(.caption)
                    .padding(.horizontal, 10)
                    .padding(.vertical, 6)
                    .background(
                        selectedCategories.contains(category)
                            ? category.color.opacity(0.2)
                            : Color.gray.opacity(0.1)
                    )
                    .foregroundStyle(
                        selectedCategories.contains(category)
                            ? category.color
                            : .gray
                    )
                    .clipShape(Capsule())
                }
            }
        }
        .padding(8)
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 16))
    }
}

// MARK: - ランドマーク詳細カード

struct LandmarkCard: View {
    let landmark: Landmark

    var body: some View {
        VStack(alignment: .leading, spacing: 6) {
            HStack {
                Image(systemName: landmark.category.iconName)
                    .foregroundStyle(landmark.category.color)
                Text(landmark.name)
                    .font(.headline)
                Spacer()
            }
            Text(landmark.description)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
        .padding()
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

#Preview {
    ContentView()
}
```


**このアプリは何をするものか：**

東京の有名な観光スポットを地図上に表示するアプリです。地図は東京をデフォルトで表示し、各スポットはカテゴリに応じた色とアイコン付きのマーカーで表示されます。画面下部のフィルターボタンをタップすることで、表示するスポットのカテゴリを切り替えることができます。ユーザーが地図をドラッグやズームして操作することで、詳細な位置情報を確認できます。

## コードの詳細解説

### データモデル（ランドマーク構造体）

```swift
struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let description: String
    let coordinate: CLLocationCoordinate2D
    let category: Category

    enum Category: String, CaseIterable {
        case temple = "寺社"
        case tower = "タワー"
        case park = "公園"

        var iconName: String {
            switch self {
            case .temple: return "building.columns"
            case .tower: return "antenna.radiowaves.left.and.right"
            case .park: return "leaf"
            }
        }

        var color: Color {
            switch self {
            case .temple: return .red
            case .tower: return .blue
            case .park: return .green
            }
        }
    }
}
```

**何をしているか：**
ランドマークという構造体を定義し、個別の観光スポットの情報を保持します。それぞれのランドマークは一意のID、名前、説明、座標、そしてカテゴリを持ちます。

**なぜこう書くのか：**
Identifiableプロトコルにより、SwiftUIが各ランドマークを一意に識別でき、ForEachで正確にマーカーを描画できます。ネストされたCategory列挙型により、カテゴリごとのアイコンと色を一箇所で管理でき、保守性が高まります。CaseIterableを採用することで、全てのカテゴリを簡単に列挙でき、フィルター機能を実装しやすくなります。

**もしこう書かなかったら：**
Identifiableを採用していなかった場合、ForEachでidパラメータを明示的に指定する必要があり、コードが冗長になります。


---

### 地図の表示とカメラ制御

```swift
@State private var cameraPosition: MapCameraPosition = .region(
    MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671),
        span: MKCoordinateSpan(latitudeDelta: 0.08, longitudeDelta: 0.08)
    )
)

var body: some View {
    ZStack(alignment: .bottom) {
        // 地図
        Map(position: $cameraPosition) {
            ForEach(filteredLandmarks) { landmark in
                Marker(
                    landmark.name,
                    systemImage: landmark.category.iconName,
                    coordinate: landmark.coordinate
                )
                .tint(landmark.category.color)
            }
        }
        .mapStyle(.standard(elevation: .realistic))
```

**何をしているか：**
cameraPositionという状態変数で地図のビューを管理します。初期値として東京の中心（緯度35.6812、経度139.7671）を設定し、ズームスパンを0.08で指定しています。Mapビューにpositionバインディングをバインドすることで、地図操作時のカメラ位置を追跡します。mapStyleで3D表示を有効化します。

**なぜこう書くのか：**
@Stateで管理することで、ユーザーが地図をドラッグやズームした時にアプリの状態が反映されます。東京の中央を初期値にすることで、ユーザーが起動時に目当ての地域をすぐに見つけやすくなります。ズームスパン0.08は東京全域をバランスよく表示できる値です。mapStyle(.standard(elevation: .realistic))により、立体的で見やすい地図表示になります。

**もしこう書かなかったら：**
もしcameraPositionを状態管理していなかった場合、ユーザーが地図を操作しても反映されず、常に同じ位置を表示するだけになります。3D表示を有効化していなかった場合、地図が平面的で分かりにくくなります。
---

### マーカーの表示

```swift
ForEach(filteredLandmarks) { landmark in
    Marker(
        landmark.name,
        systemImage: landmark.category.iconName,
        coordinate: landmark.coordinate
    )
    .tint(landmark.category.color)
}
```

**何をしているか：**
フィルター済みのランドマークリストをループし、各ランドマークのマーカーを地図上に配置します。`Marker`にはスポット名、カテゴリのアイコン（SF Symbols）、座標を指定し、`.tint`修飾子でカテゴリに対応した色でマーカーを着色します。

**なぜこう書くのか：**
systemImageで標準アイコンを使用することで、外部の画像ファイルが不要になり、アプリサイズが軽くなります。tintでカテゴリごとに色分けすることで、ユーザーが視覚的にスポットの種類を即座に識別できます。フィルター済みのランドマークのみ表示することで、ユーザーの選択が正確に反映されます。

**もしこう書かなかったら：**
カテゴリ色を指定していなかった場合、全てのマーカーが同じ色になり、スポットの種類が区別しにくくなります。filteredLandmarksではなくsampleDataをそのまま使った場合、フィルター機能が機能しません。
---

### フィルター機能

```swift
@State private var selectedCategories: Set<Landmark.Category> = Set(Landmark.Category.allCases)

var filteredLandmarks: [Landmark] {
    Landmark.sampleData.filter { selectedCategories.contains($0.category) }
}

struct CategoryFilter: View {
    @Binding var selectedCategories: Set<Landmark.Category>

    var body: some View {
        HStack(spacing: 8) {
            ForEach(Landmark.Category.allCases, id: \.self) { category in
                Button {
                    if selectedCategories.contains(category) {
                        selectedCategories.remove(category)
                    } else {
                        selectedCategories.insert(category)
                    }
                } label: {
                    HStack(spacing: 4) {
                        Image(systemName: category.iconName)
                        Text(category.rawValue)
                    }
                    .font(.caption)
                    .padding(.horizontal, 10)
                    .padding(.vertical, 6)
                    .background(
                        selectedCategories.contains(category)
                            ? category.color.opacity(0.2)
                            : Color.gray.opacity(0.1)
                    )
                    .foregroundStyle(
                        selectedCategories.contains(category)
                            ? category.color
                            : .gray
                    )
                    .clipShape(Capsule())
                }
            }
        }
        .padding(8)
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 16))
    }
}
```

**何をしているか：**

selectedCategoriesという状態変数で現在選択されているカテゴリをSetで管理します。filteredLandmarks計算プロパティで、選択されたカテゴリに属するランドマークのみをフィルタリングします。CategoryFilterビューでは、全てのカテゴリをボタンで表示し、タップするとカテゴリを追加または削除できます。選択状態に応じてボタンの背景色と文字色が変わります。

**なぜこう書くのか：**

Setを使用することで、重複がなく、追加・削除操作が効率的です（配列の場合は毎回線形探索が必要）。計算プロパティfilteredLandmarksにより、状態が変わるたびに自動的にマーカーが更新されます。ボタンの選択状態を視覚的に区別することで、ユーザーが現在の選択状態を直感的に理解できます。

**もしこう書かなかったら：**

もしselectedCategoriesを配列で管理していた場合、同じカテゴリが複数登録される可能性があり、処理が複雑になります。フィルター機能がなかった場合、6つ全てのマーカーが常に表示され、画面が見づらくなります。ボタンの見た目が選択状態を反映していなかった場合、ユーザーが現在の選択状態を把握しにくくなります。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| `Map` | SwiftUIで地図を表示するビューコンポーネント | `Map(position: $cameraPosition) { ... }` |
| `Marker` | 地図上に位置をマーキングするコンポーネント | `Marker(landmark.name, systemImage: landmark.category.iconName, coordinate: landmark.coordinate)` |
| `MapCameraPosition` | 地図のカメラ（表示位置）を制御する型 | `.region(MKCoordinateRegion(...))` |
| `MKCoordinateRegion` | 地図領域を定義（中心とズームスパン） | `MKCoordinateRegion(center: CLLocationCoordinate2D(...), span: MKCoordinateSpan(...))` |
| `CLLocationCoordinate2D` | 緯度経度を表現する型 | `CLLocationCoordinate2D(latitude: 35.6812, longitude: 139.7671)` |
| `Identifiable` | プロトコル：オブジェクトに一意IDを付与 | `struct Landmark: Identifiable { let id = UUID() }` |
| `CaseIterable` | 列挙型の全ケースを列挙可能にする | `enum Category: CaseIterable { ... }` で `Category.allCases` が使用可能 |
| `@Binding` | 親ビューの状態を子ビューで編集可能に | `@Binding var selectedCategories: Set<Landmark.Category>` |
| `Set<T>` | 重複のない順序不定のコレクション | `Set(Landmark.Category.allCases)` |
| `mapStyle` | 地図のスタイルを設定 | `.mapStyle(.standard(elevation: .realistic))` |
| `systemImage` | SF Symbolsの標準アイコンを使用 | `Image(systemName: "building.columns")` |
| `CaseIterable` + `allCases` | 列挙型の全ケースを配列で取得 | `ForEach(Landmark.Category.allCases) { ... }` |

---

## 自分の実験メモ

（模範コードを改変して試したことを書く）

**実験1： **
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
