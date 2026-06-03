# 第3章：カメラの利用

> 執筆者：　アントニオ
> 最終更新：2026-05-20

## この章で学ぶこと

iPhoneのカメラとフォトライブラリにアクセスしてSwiftUIアプリで画像を表示する方法を学ぶ。UIKitのカメラピッカーをSwiftUIアプリにUIViewControllerRepresentableと、UIKitのイベント処理をSwiftUIの状態管理に繋げるためのCoordinatorを理解することが重要である。

## 模範コードの全体像

（教員から配布された模範コードをここに貼り付ける）

```swift
// MARK: - メインビュー

struct ContentView: View {
    @State private var selectedItem: PhotosPickerItem?
    @State private var selectedImage: Image?
    @State private var isShowingCamera = false
    @State private var capturedUIImage: UIImage?

    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // 画像表示エリア
                imageDisplayArea

                // ボタンエリア
                HStack(spacing: 20) {
                    // フォトライブラリから選択
                    PhotosPicker(selection: $selectedItem, matching: .images) {
                        Label("ライブラリ", systemImage: "photo.on.rectangle")
                    }
                    .buttonStyle(.bordered)

                    // カメラで撮影（シミュレータには未搭載のため自動的に無効化）
                    Button {
                        isShowingCamera = true
                    } label: {
                        Label("カメラ", systemImage: "camera")
                    }
                    .buttonStyle(.borderedProminent)
                    .disabled(!UIImagePickerController.isSourceTypeAvailable(.camera))
                }
                .padding()
            }
            .navigationTitle("写真アプリ")
            .onChange(of: selectedItem) { _, newItem in
                Task {
                    await loadImage(from: newItem)
                }
            }
            .fullScreenCover(isPresented: $isShowingCamera) {
                CameraView(capturedImage: $capturedUIImage)
            }
            .onChange(of: capturedUIImage) { _, newImage in
                if let uiImage = newImage {
                    selectedImage = Image(uiImage: uiImage)
                }
            }
        }
    }

    // MARK: - 画像表示エリア

    @ViewBuilder
    private var imageDisplayArea: some View {
        if let image = selectedImage {
            image
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(maxHeight: 400)
                .clipShape(RoundedRectangle(cornerRadius: 16))
                .shadow(radius: 4)
                .padding()
        } else {
            RoundedRectangle(cornerRadius: 16)
                .fill(.gray.opacity(0.1))
                .frame(height: 300)
                .overlay {
                    VStack(spacing: 8) {
                        Image(systemName: "photo")
                            .font(.system(size: 48))
                            .foregroundStyle(.gray)
                        Text("写真を選択または撮影してください")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                    }
                }
                .padding()
        }
    }

    // MARK: - 画像の読み込み

    func loadImage(from item: PhotosPickerItem?) async {
        guard let item = item else { return }

        do {
            if let data = try await item.loadTransferable(type: Data.self),
               let uiImage = UIImage(data: data) {
                selectedImage = Image(uiImage: uiImage)
            }
        } catch {
            print("画像の読み込みに失敗: \(error.localizedDescription)")
        }
    }
}

// MARK: - カメラビュー（UIKit連携）

struct CameraView: UIViewControllerRepresentable {
    @Binding var capturedImage: UIImage?
    @Environment(\.dismiss) private var dismiss

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.sourceType = .camera
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: CameraView

        init(_ parent: CameraView) {
            self.parent = parent
        }

        func imagePickerController(
            _ picker: UIImagePickerController,
            didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
        ) {
            if let image = info[.originalImage] as? UIImage {
                parent.capturedImage = image
            }
            parent.dismiss()
        }

        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.dismiss()
        }
    }
}

#Preview {
    ContentView()
}

```

**このアプリは何をするものか：**

写真を2つの方法で取得して画面に表示する機能を持つシンプルな写真アプリです。ユーザーは「ライブラリ」ボタンをタップしてiPhoneのフォトライブラリから既存の写真を選択するか、「カメラ」ボタンをタップしてその場でカメラを起動して新しい写真を撮影できます。どちらの方法で取得した写真でも、画面中央の大きな領域に表示されます。撮影デバイスがカメラを持っていない場合（シミュレータなど）は、カメラボタンが自動的に無効化されます。

## コードの詳細解説

### PhotosPickerによる写真選択

```swift
PhotosPicker(selection: $selectedItem, matching: .images) {
    Label("ライブラリ", systemImage: "photo.on.rectangle")
}
.buttonStyle(.bordered)
```

**何をしているか：**

PhotosPickerコンポーネントを使用してフォトライブラリのアクセスインターフェースを実装しています。$selectedItemというバインディングを通じて、ユーザーが選択した写真の情報を状態変数に保存します。matching: .imagesフィルタによって、画像ファイルのみが表示されるようにしています。クロージャー内のLabelは、ユーザーが見るボタンの表示内容を定義しています。

**なぜこう書くのか：**

PhotosPickerを使うことで、Apple公式のフォトライブラリUIを利用でき、ユーザー体験が統一され、プライバシー権限も自動的に処理されます。古い方法のUIImagePickerControllerでフォトライブラリを使う場合、廃止予定の警告が出るため、SwiftUI環境では推奨されません。$selectedItemでバインディングすることで、ユーザーの選択が自動的に反映され、@onChangeで変化を監視して画像読み込みを開始できます。

**もしこう書かなかったら：**

もしPhotosPicker を使わずにUIImagePickerControllerでフォトライブラリにアクセスしようとした場合、廃止予定警告が出ます。もしselection バインディングを使わずに単独のボタンにしていたら、ユーザーが選択した結果をSwiftUIビューに反映させるコードが複雑になり、CoordinatorパターンをUIImagePickerControllerの代わりに毎回実装する必要が生じます。

---

### 画像の非同期読み込み

```swift
func loadImage(from item: PhotosPickerItem?) async {
    guard let item = item else { return }

    do {
        if let data = try await item.loadTransferable(type: Data.self),
           let uiImage = UIImage(data: data) {
            selectedImage = Image(uiImage: uiImage)
        }
    } catch {
        print("画像の読み込みに失敗: \(error.localizedDescription)")
    }
}
```

**何をしているか：**

PhotosPickerItemから画像のバイナリデータを非同期で読み込み、それをUIImageに変換し、最後にSwiftUIの Image に変換して状態変数に保存しています。guard文で項目がnilでないか確認し、do-catchブロックで読み込み処理のエラー処理も行っています。

**なぜこう書くのか：**

画像ファイルの読み込みはネットワークアクセスやディスクIOを含むため、メインスレッドをブロックしないよう asyncキーワードを使用します。awaitで非同期操作の完了を待ちます。Data.self で汎用的なバイナリデータとして取得してからUIImageに変換することで、JPEG、PNG、HEICなど複数の画像形式に対応できます。エラーハンドリングにより、読み込み失敗時にアプリがクラッシュせず、デバッグ情報がコンソールに出力されます。

**もしこう書かなかったら：**

awaitを使わず同期的に読み込もうとしたら、読み込み中にUIが応答しなくなってアプリが「応答なし」状態になります。もしguard文がなかったら、nilの項目をアンラップしようとしてクラッシュします。もしエラーハンドリングを省いたら、読み込み失敗時にクラッシュするか、沈黙のまま失敗します。もしUIImageを経由せずにDataから直接Imageを作成しようとしたら、SwiftUIのImageはバイナリDataダイレクトに対応していないため、コンパイルエラーになります。

---

### UIViewControllerRepresentableによるカメラ連携

```swift
struct CameraView: UIViewControllerRepresentable {
    @Binding var capturedImage: UIImage?
    @Environment(\.dismiss) private var dismiss

    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.sourceType = .camera
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
}
```

**何をしているか：**

UIViewControllerRepresentableプロトコルを実装してUIImagePickerControllerをSwiftUIラッパーで囲み、SwiftUIビュー階層に組み込めるようにしています。makeUIViewControllerメソッドでカメラモードのpickerを作成し、delegateに自分のCoordinatorを設定して、イベント処理を委譲しています。@Bindingで親ビューとの双方向バインディングを行い、capturedImageが更新されます。@Environmentで dismiss環境値を取得し、Coordinatorから画像選択後にビューを閉じるために使用します。

**なぜこう書くのか：**

SwiftUIはネイティブにカメラUIを持たないため、UIKitの UIImagePickerControllerを使う必要があります。UIViewControllerRepresentableはUIKitコンポーネントをSwiftUIにブリッジするために存在します。delegateパターンをサポートするため、Coordinatorを通じてイベントを受け取ります。makeCoordinatorメソッドで Coordinator インスタンスを作成することで、親ビューへのアクセスと状態更新が可能になります。updateUIViewControllerは毎フレーム呼ばれるため、ここに定期的な更新処理があればそこに書きます。

**もしこう書かなかったら：**

UIViewControllerRepresentableを使わずにUIImagePickerControllerを直接使おうとしたら、SwiftUIビュー構造体内で使用できず、コンパイルエラーになります。もしdelegateを設定しなかったら、ユーザーが写真を選択してもそのイベントが通知されず、撮影結果を扱えません。もし@Bindingを使わなかったら、選択画像を親ビューに返す方法がなくなります。もしdismiss環境値を取得しなかったら、撮影後に自動的に画面を閉じられません。

---

### Coordinatorパターン

```swift
class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
    let parent: CameraView

    init(_ parent: CameraView) {
        self.parent = parent
    }

    func imagePickerController(
        _ picker: UIImagePickerController,
        didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
    ) {
        if let image = info[.originalImage] as? UIImage {
            parent.capturedImage = image
        }
        parent.dismiss()
    }

    func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
        parent.dismiss()
    }
}
```

**何をしているか：**

UIImagePickerControllerのdelegate メソッドを実装して、カメライベントに応答するクラスです。ユーザーが写真を撮影して「使用」をタップした時の didFinishPickingMediaWithInfo と、キャンセルした時の imagePickerControllerDidCancel の2つのメソッドを実装しています。parent プロパティで親のCameraViewを参照し、parentの@Bindingである capturedImage に撮影結果を代入することで、親ビューに画像が通知されます。両ケースでparent.dismiss()を呼び出してカメラUIを閉じます。

**なぜこう書くのか：**

UIImagePickerControllerはdelegateパターンでイベント通知を行うため、delegateプロトコルを実装したクラスが必要です。NSObjectを継承しているのは、UIKitのdelegateシステムがObjective-Cランタイムに依存しているためです。parentを保持することで、CoordinatorからSwiftUIの状態（@Binding）にアクセスできます。didFinishPickingMediaWithInfoでは、infoディクショナリからキーを指定して画像を取り出す必要があります。.originalImageキーを使うのは、加工前の元の撮影画像を取得するためです。両ケースでdismissを呼び出すことで、どちらのシナリオ（撮影完了 or キャンセル）でも画面が自動的に閉じます。

**もしこう書かなかったら：**

もしCoordinatorクラスを作らなかったら、UIImagePickerControllerのdelegateメソッドを実装する場所がなく、イベント通知を受けられません。もしNSObjectを継承しなかったら、Objective-Cランタイムに認識されず、delegateメソッドが呼ばれません。もしparentを保持していなかったら、撮影結果をSwiftUIビューの状態に渡す方法がなくなり、画像が表示されません。もしdismissを呼ばなかったら、撮影後もカメラUIが表示されたままになり、ユーザーが手動でキャンセルボタンを押す必要があります。

---

（必要に応じてセクションを増やす）

## 新しく学んだSwiftの文法・API

| 項目 | 説明 | 使用例 |
|------|------|--------|
| `PhotosPicker` | フォトライブラリから画像を選択するSwiftUIコンポーネント | `PhotosPicker(selection: $selectedItem, matching: .images)` |
| `PhotosPickerItem` | PhotosPickerで選択されたアイテムを表す型 | `selectedItem: PhotosPickerItem?` |
| `UIImagePickerController` | カメラまたはフォトライブラリにアクセスするUIKitコンポーネント | `picker.sourceType = .camera` |
| `UIViewControllerRepresentable` | UIKitのUIViewControllerをSwiftUIで使うためのプロトコル | `struct CameraView: UIViewControllerRepresentable` |
| `@Environment(\.dismiss)` | ビューを閉じるための環境値 | `@Environment(\.dismiss) private var dismiss` |
| `fullScreenCover` | モーダルでビューを全画面表示するモディファイア | `.fullScreenCover(isPresented: $isShowingCamera)` |
| `async/await` | 非同期処理を扱うSwiftの構文 | `func loadImage(from item: PhotosPickerItem?) async` |
| `makeCoordinator()` | UIViewControllerRepresentableで委譲オブジェクトを作成するメソッド | `func makeCoordinator() -> Coordinator` |
| `Coordinator パターン` | UIKitのdelegateをSwiftUIブリッジする設計パターン | `class Coordinator: NSObject, UIImagePickerControllerDelegate` |
| `@ViewBuilder` | 複数のビューを条件付きで返すためのプロパティビルダー | `@ViewBuilder private var imageDisplayArea: some View` |

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
