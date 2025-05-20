---
title: "Flutterを用いたMapLibreの基本的な機能の実装方法"
emoji: "🗺️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "MapLibre", "地図", "モバイル開発"]
published: false
---

## ゴール（はじめに）：Flutterアプリに地図機能を実装する

この記事では、FlutterアプリにMapLibreを組み込んで、基本的な地図機能（地図表示、マーカー設置、ユーザー位置表示など）を実装する方法について解説します。MapLibreはオープンソースの地図ライブラリで、商用利用も無料で行えるため、地図機能を手軽に実装したい開発者に最適です。

## やってみた結果

この記事を読むことで、以下のことが実現できるようになります：
- Flutterアプリに高性能な地図を表示する
- 地図上にマーカーやポリゴンなどを配置する
- ユーザーの現在位置を地図上に表示する
- 地図のスタイルをカスタマイズする

## 開発環境
- Flutter 3.19.0以上
- Dart 3.3.0以上
- Android Studio / VS Code
- iOS / Androidエミュレータ

## 事前準備

MapLibreをFlutterで使用するには、以下のパッケージが必要です：
- `maplibre_gl`: MapLibreのFlutterプラグイン

## やったこと

この記事では、以下のステップに沿って進めていきます：

1. MapLibre GLプラグインのインストール
2. プロジェクトの設定
3. 基本的な地図表示
4. 地図のカスタマイズ（スタイル変更）
5. マーカーの追加
6. ユーザー位置の表示
7. 地図のインタラクション制御

## MapLibre GLのインストール

まず、プロジェクトにMapLibre GLプラグインをインストールします。`pubspec.yaml`ファイルに以下の依存関係を追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  maplibre_gl: ^0.18.0
  location: ^4.4.0  # ユーザー位置取得用
```

追加したら、ターミナルで次のコマンドを実行して依存関係をインストールします：

```bash
flutter pub get
```

## プロジェクトの設定

### Android設定

Android用の設定を行うため、`android/app/build.gradle`ファイルを開き、以下の設定を追加します：

```gradle
android {
    defaultConfig {
        // 他の設定...
        minSdkVersion 20
    }
}
```

### iOS設定

iOS用の設定を行うために、`ios/Runner/Info.plist`ファイルに位置情報アクセス許可の説明を追加します：

```xml
<dict>
    <!-- 他の設定... -->
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>このアプリはマップ上にあなたの位置を表示するために位置情報へのアクセスが必要です。</string>
    <key>io.flutter.embedded_views_preview</key>
    <true/>
</dict>
```

## 基本的な地図表示

まず、基本的な地図を表示する簡単なアプリを作成します。以下のコードを`main.dart`に追加します：

```dart
import 'package:flutter/material.dart';
import 'package:maplibre_gl/maplibre_gl.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'MapLibre Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MapScreen(),
    );
  }
}

class MapScreen extends StatefulWidget {
  @override
  _MapScreenState createState() => _MapScreenState();
}

class _MapScreenState extends State<MapScreen> {
  MaplibreMapController? mapController;
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('MapLibre実装例'),
      ),
      body: MaplibreMap(
        styleString: 'https://demotiles.maplibre.org/style.json',
        initialCameraPosition: CameraPosition(
          target: LatLng(35.681236, 139.767125), // 東京駅
          zoom: 14.0,
        ),
        onMapCreated: (controller) {
          mapController = controller;
        },
      ),
    );
  }
}
```

このコードで、東京駅を中心にした基本的な地図が表示されます。`styleString`には、MapLibreで利用可能な地図スタイルのURLを指定しています。

## 地図のスタイルカスタマイズ

MapLibreでは様々な地図スタイルを使用できます。以下は人気のあるいくつかのスタイルです：

- OpenStreetMap標準: `https://demotiles.maplibre.org/style.json`
- Maptiler Basic: `https://api.maptiler.com/maps/basic/style.json?key=YOUR_MAPTILER_KEY`
- OpenMapTiles: `https://openmaptiles.github.io/osm-bright-gl-style/style-cdn.json`

スタイルを変更するボタンを追加するには、以下のようにコードを修正します：

```dart
// _MapScreenStateクラス内に追加
void _changeMapStyle(String styleUrl) {
  mapController?.setStyleString(styleUrl);
}

@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text('MapLibre実装例'),
    ),
    body: Stack(
      children: [
        MaplibreMap(
          styleString: 'https://demotiles.maplibre.org/style.json',
          initialCameraPosition: CameraPosition(
            target: LatLng(35.681236, 139.767125), // 東京駅
            zoom: 14.0,
          ),
          onMapCreated: (controller) {
            mapController = controller;
          },
        ),
        Positioned(
          top: 10,
          right: 10,
          child: Column(
            children: [
              ElevatedButton(
                onPressed: () => _changeMapStyle('https://demotiles.maplibre.org/style.json'),
                child: Text('標準スタイル'),
              ),
              SizedBox(height: 10),
              ElevatedButton(
                onPressed: () => _changeMapStyle('https://openmaptiles.github.io/osm-bright-gl-style/style-cdn.json'),
                child: Text('明るいスタイル'),
              ),
            ],
          ),
        ),
      ],
    ),
  );
}
```

## マーカーの追加

地図上にマーカーを追加するには、以下のように実装します：

```dart
// _MapScreenStateクラス内に追加
final List<Symbol> _symbols = [];

Future<void> _addMarker(LatLng position, String title) async {
  final symbol = await mapController?.addSymbol(
    SymbolOptions(
      geometry: position,
      iconImage: 'marker-15', // デフォルトのマーカーアイコン
      iconSize: 2.0,
      textField: title,
      textOffset: Offset(0, 1.5),
      textSize: 12.0,
    ),
  );
  
  if (symbol != null) {
    setState(() {
      _symbols.add(symbol);
    });
  }
}
```

この機能を使用するために、地図上をタップしたときにマーカーを配置するよう実装します：

```dart
MaplibreMap(
  styleString: 'https://demotiles.maplibre.org/style.json',
  initialCameraPosition: CameraPosition(
    target: LatLng(35.681236, 139.767125), // 東京駅
    zoom: 14.0,
  ),
  onMapCreated: (controller) {
    mapController = controller;
  },
  onMapClick: (point, latLng) {
    _addMarker(latLng, '新しいマーカー');
  },
),
```

## ユーザー位置の表示

ユーザーの現在位置を地図上に表示するには、まず`location`パッケージを使って位置情報を取得し、その後地図を更新します：

```dart
// importに追加
import 'package:location/location.dart';

// _MapScreenStateクラス内に追加
Location location = Location();
bool _serviceEnabled = false;
PermissionStatus _permissionGranted = PermissionStatus.denied;

Future<void> _initLocationService() async {
  _serviceEnabled = await location.serviceEnabled();
  if (!_serviceEnabled) {
    _serviceEnabled = await location.requestService();
    if (!_serviceEnabled) {
      return;
    }
  }

  _permissionGranted = await location.hasPermission();
  if (_permissionGranted == PermissionStatus.denied) {
    _permissionGranted = await location.requestPermission();
    if (_permissionGranted != PermissionStatus.granted) {
      return;
    }
  }

  // 現在位置を取得して地図を移動
  final locationData = await location.getLocation();
  final latLng = LatLng(locationData.latitude!, locationData.longitude!);
  
  await mapController?.animateCamera(
    CameraUpdate.newLatLngZoom(latLng, 15.0),
  );
  
  // 現在位置にマーカーを追加
  await _addMarker(latLng, '現在地');
  
  // 位置情報の更新を監視
  location.onLocationChanged.listen((LocationData currentLocation) {
    if (mapController != null) {
      mapController!.animateCamera(
        CameraUpdate.newLatLng(
          LatLng(currentLocation.latitude!, currentLocation.longitude!),
        ),
      );
    }
  });
}

@override
void initState() {
  super.initState();
  // アプリ起動時に位置情報サービスを初期化
  Future.delayed(Duration.zero, () {
    _initLocationService();
  });
}
```

## 地図のインタラクション制御

地図のズーム、回転、チルトなどを制御するためのボタンを追加します：

```dart
// StackのChildrenに追加
Positioned(
  bottom: 20,
  right: 20,
  child: Column(
    children: [
      FloatingActionButton(
        heroTag: "zoom_in",
        child: Icon(Icons.add),
        onPressed: () {
          mapController?.animateCamera(
            CameraUpdate.zoomIn(),
          );
        },
      ),
      SizedBox(height: 10),
      FloatingActionButton(
        heroTag: "zoom_out",
        child: Icon(Icons.remove),
        onPressed: () {
          mapController?.animateCamera(
            CameraUpdate.zoomOut(),
          );
        },
      ),
      SizedBox(height: 10),
      FloatingActionButton(
        heroTag: "current_location",
        child: Icon(Icons.my_location),
        onPressed: _initLocationService,
      ),
    ],
  ),
),
```

## 結論に対しての補足

- **カスタムマーカー画像の使用**: 独自のマーカーアイコンを使用するには、アセットを追加し、`mapController?.addImage()`メソッドを使って登録した後に使用します。

- **オフライン地図**: MapLibreはオフライン地図をサポートしています。詳細は[公式ドキュメント](https://github.com/maplibre/maplibre-gl-native)を参照してください。

- **パフォーマンス最適化**: 多数のマーカーを表示する場合は、通常のマーカーの代わりにクラスタリングを検討してください。

- **代替サービス**: 商用アプリで高度な機能が必要な場合は、Google MapsやMapboxなどの有料サービスも検討してみてください。

## おわりに

この記事では、FlutterアプリにMapLibreを統合して基本的な地図機能を実装する方法について解説しました。MapLibreは無料で使えるオープンソースの地図ソリューションであり、基本的な地図機能を実装するのに十分な機能を備えています。

今回紹介した基本機能を応用すれば、位置情報ベースのサービスやナビゲーションアプリなど、様々な地図関連アプリケーションを開発できるでしょう。ぜひ実際に試してみてください！

## 参考リンク

- [MapLibre GL公式ドキュメント](https://github.com/maplibre/maplibre-gl-native)
- [Flutter MapLibre GL Plugin](https://github.com/maplibre/flutter-maplibre-gl)
- [OpenMapTilesプロジェクト](https://openmaptiles.org/)
