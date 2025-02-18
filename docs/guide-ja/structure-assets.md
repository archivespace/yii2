アセット
========

Yii では、アセットは、ウェブ・ページで参照できるファイルを意味します。アセットは CSS ファイルであったり、JavaScript ファイルであったり、画像やビデオのファイルであったりします。
アセットはウェブでアクセス可能なディレクトリに配置され、ウェブ・サーバによって直接に提供されます。

たいていの場合、アセットはプログラム的に管理する方が望ましいものです。
例えば、ページの中で [[yii\jui\DatePicker]] ウィジェットを使うとき、このウィジェットは必要な CSS と JavaScript のファイルを自動的にインクルードします。
あなたに対して、手作業で必要なファイルを探してインクルードするように要求したりはしません。
そして、このウィジェットを新しいバージョンにアップグレードしたときは、自動的に新しいバージョンのアセット・ファイルが使用されるようになります。
このチュートリアルでは、Yii によって提供される強力なアセット管理機能について説明します。


## アセット・バンドル <span id="asset-bundles"></span>

Yii はアセットを *アセット・バンドル* を単位として管理します。アセット・バンドルは、簡単に言えば、
あるディレクトリの下に集められた一群のアセットです。
[ビュー](structure-views.md) の中でアセット・バンドルを登録すると、バンドルの中の CSS や JavaScript のファイルがレンダリングされるウェブ・ページに挿入されます。


## アセット・バンドルを定義する <span id="defining-asset-bundles"></span>

アセット・バンドルは [[yii\web\AssetBundle]] から拡張された PHP クラスとして定義されます。
バンドルの名前は、対応する PHP クラスの完全修飾名 (先頭のバック・スラッシュを除く) です。
アセット・バンドル・クラスは [オートロード可能](concept-autoloading.md) でなければなりません。
アセット・バンドル・クラスは、通常、アセットがどこに置かれているか、バンドルがどういう CSS や JavaScript のファイルを含んでいるか、そして、バンドルが他のバンドルにどのように依存しているかを定義します。

以下のコードは [ベーシック・プロジェクト・テンプレート](start-installation.md) によって使用されているメインのアセット・バンドルを定義するものです。

```php
<?php

namespace app\assets;

use yii\web\AssetBundle;

class AppAsset extends AssetBundle
{
    public $basePath = '@webroot';
    public $baseUrl = '@web';
    public $css = [
        'css/site.css',
        ['css/print.css', 'media' => 'print'],
    ];
    public $js = [
    ];
    public $depends = [
        'yii\web\YiiAsset',
        'yii\bootstrap\BootstrapAsset',
    ];
}
```

上の `AppAsset` クラスは、アセット・ファイルが `@webroot` ディレクトリの下に配置されており、それが URL `@web` に対応することを定義しています。
バンドルは一つだけ CSS ファイル `css/site.css` を含み、JavaScript ファイルは含みません。
バンドルは、他の二つのバンドル、すなわち [[yii\web\YiiAsset]] と [[yii\bootstrap\BootstrapAsset]] に依存しています。
以下、[[yii\web\AssetBundle]] のプロパティに関して、更に詳細に説明します。

* [[yii\web\AssetBundle::sourcePath|sourcePath]]: このバンドルのアセット・ファイルを含むルート・ディレクトリを指定します。
  ルート・ディレクトリがウェブ・アクセス可能でない場合に、このプロパティをセットしなければなりません。
  そうでない場合は、代りに、[[yii\web\AssetBundle::basePath|basePath]] と [[yii\web\AssetBundle::baseUrl|baseUrl]] のプロパティをセットしなければなりません。
  [パス・エイリアス](concept-aliases.md) をここで使うことが出来ます。
* [[yii\web\AssetBundle::basePath|basePath]]: このバンドルのアセット・ファイルを含むウェブ・アクセス可能なディレクトリを指定します。
  [[yii\web\AssetBundle::sourcePath|sourcePath]] プロパティをセットした場合は、[アセット・マネージャ](#asset-manager) がバンドルに含まれるファイルをウェブ・アクセス可能なディレクトリに発行して、
  その結果、このプロパティを上書きします。
  アセット・ファイルが既にウェブ・アクセス可能なディレクトリにあり、アセットの発行が必要でない場合に、このプロパティをセットしなければなりません。
  [パス・エイリアス](concept-aliases.md) をここで使うことが出来ます。
* [[yii\web\AssetBundle::baseUrl|baseUrl]]: [[yii\web\AssetBundle::basePath|basePath]] ディレクトリに対応する URL を指定します。
  [[yii\web\AssetBundle::basePath|basePath]] と同じように、[[yii\web\AssetBundle::sourcePath|sourcePath]] プロパティをセットした場合は、
  [アセット・マネージャ](#asset-manager) がアセットを発行して、その結果、このプロパティを上書きします。
  [パス・エイリアス](concept-aliases.md) をここで使うことが出来ます。
* [[yii\web\AssetBundle::css|css]]: このバンドルに含まれている CSS ファイルをリストする配列です。
  ディレクトリの区切りとしてフォワード・スラッシュ "/" だけを使わなければならないことに注意してください。
  それぞれのファイルは、個別に、パス文字列、または、パス文字列と属性のタグと値を一緒に含む配列によって指定することが出来ます。
* [[yii\web\AssetBundle::js|js]]: このバンドルに含まれる JavaScript ファイルをリストする配列です。
  この配列の形式は、[[yii\web\AssetBundle::css|css]] の配列の形式と同じです。
  それぞれの JavaScript ファイルは、以下の二つの形式のどちらかによって指定することが出来ます。
  - ローカルの JavaScript ファイルを表す相対パス (例えば `js/main.js`)。
    実際のファイルのパスは、この相対パスの前に [[yii\web\AssetManager::basePath]] を付けることによって決定されます。
    また、実際の URL は、この相対パスの前に [[yii\web\AssetManager::baseUrl]] を付けることによって決定されます。
  - 外部の JavaScript ファイルを表す絶対 URL。
    例えば、`https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js` や
    `//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js` など。
* [[yii\web\AssetBundle::depends|depends]]: このバンドルが依存しているアセット・バンドルの名前をリストする配列です
   (バンドルの依存関係については、すぐ後で説明します)。
* [[yii\web\AssetBundle::jsOptions|jsOptions]]: このバンドルにある *全て* の JavaScript ファイルについて、
  それを登録するときに呼ばれる [[yii\web\View::registerJsFile()]] メソッドに渡されるオプションを指定します。
* [[yii\web\AssetBundle::cssOptions|cssOptions]]: このバンドルにある *全て* の CSS ファイルについて、
  それを登録するときに呼ばれる [[yii\web\View::registerCssFile()]] メソッドに渡されるオプションを指定します。
* [[yii\web\AssetBundle::publishOptions|publishOptions]]: ソースのアセット・ファイルをウェブ・ディレクトリに発行するときに呼ばれる
  [[yii\web\AssetManager::publish()]] メソッドに渡されるオプションを指定します。
  これは [[yii\web\AssetBundle::sourcePath|sourcePath]] プロパティを指定した場合にだけ使用されます。


### アセットの配置場所 <span id="asset-locations"></span>

アセットは、配置場所を基準にして、次のように分類することが出来ます。

* ソース・アセット: アセット・ファイルは、ウェブ経由で直接にアクセスすることが出来ない PHP ソース・コードと一緒に配置されています。
  ページの中でソース・アセットを使用するためには、ウェブ・ディレクトリにコピーして、いわゆる発行されたアセットに変換しなければなりません。
  このプロセスは、すぐ後で詳しく説明しますが、*アセット発行* と呼ばれます。
* 発行されたアセット: アセット・ファイルはウェブ・ディレクトリに配置されており、したがってウェブ経由で直接にアクセスすることが出来ます。
* 外部アセット: アセット・ファイルは、あなたのウェブ・アプリケーションをホストしているのとは別のウェブ・サーバ上に
  配置されています。

アセット・バンドル・クラスを定義するときに、[[yii\web\AssetBundle::sourcePath|sourcePath]] プロパティを指定した場合は、
相対パスを使ってリストに挙げられたアセットは全てソース・アセットであると見なされます。
このプロパティを指定しなかった場合は、アセットは発行されたアセットであることになります
(したがって、[[yii\web\AssetBundle::basePath|basePath]] と [[yii\web\AssetBundle::baseUrl|baseUrl]] を指定して、アセットがどこに配置されているかを Yii に知らせなければなりません)。

アプリケーションに属するアセットは、不要なアセット発行プロセスを避けるために、ウェブ・ディレクトリに置くことが推奨されます。
前述の例において `AppAsset` が [[yii\web\AssetBundle::sourcePath|sourcePath]] ではなく
[[yii\web\AssetBundle::basePath|basePath]] を指定しているのは、これが理由です。

[エクステンション](structure-extensions.md) の場合は、
アセットがソース・コードと一緒にウェブからアクセス出来ないディレクトリに配置されているため、
アセット・バンドル・クラスを定義するときには [[yii\web\AssetBundle::sourcePath|sourcePath]] プロパティを指定しなければなりません。

> Note: `@webroot/assets` を [[yii\web\AssetBundle::sourcePath|ソース・パス]] として使ってはいけません。
  このディレクトリは、デフォルトでは、[[yii\web\AssetManager|アセット・マネージャ]] がソースの配置場所から発行されたアセット・ファイルを保存する場所として使われます。
  このディレクトリの中のファイルはすべて一時的なものと見なされており、
  削除されることがあります。


### アセットの依存関係 <span id="asset-dependencies"></span>

ウェブ・ページに複数の CSS や JavaScript ファイルをインクルードするときは、オーバーライドの問題を避けるために、一定の順序に従わなければなりません。
例えば、ウェブ・ページで jQuery UI ウィジェットを使おうとするときは、jQuery JavaScript ファイルが
jQuery UI JavaScript ファイルより前にインクルードされることを保証しなければなりません。
このような順序付けをアセット間の依存関係と呼びます。

アセットの依存関係は、主として、[[yii\web\AssetBundle::depends]] プロパティによって指定されます。
`AppAsset` の例では、このアセット・バンドルは他の二つのアセット・バンドル、すなわち、[[yii\web\YiiAsset]] と [[yii\bootstrap\BootstrapAsset]] に依存しています。
このことは、`AppAsset` の CSS と JavaScript ファイルが、依存している二つのアセット・バンドルにあるファイルの *後に*
インクルードされることを意味します。

アセットの依存関係は中継されます。つまり、バンドル A が B に依存し、B が C に依存していると、A は C にも依存していることになります。


### アセットのオプション <span id="asset-options"></span>

[[yii\web\AssetBundle::cssOptions|cssOptions]] および [[yii\web\AssetBundle::jsOptions|jsOptions]] のプロパティを指定して、
CSS と JavaScript ファイルがページにインクルードされる方法をカスタマイズすることが出来ます。
これらのプロパティの値は、[ビュー](structure-views.md) が CSS と JavaScript ファイルをインクルードするために、[[yii\web\View::registerCssFile()]] および
[[yii\web\View::registerJsFile()]] メソッドを呼ぶときに、それぞれ、オプションとして引き渡されます。

> Note: バンドル・クラスでセットしたオプションは、バンドルの中の *全て* の CSS/JavaScript ファイルに適用されます。
  いろいろなファイルに別々のオプションを使用したい場合は、上述した [[yii\web\AssetBundle::css|css] の形式を使うか、
または、別々のアセット・バンドルを作成して、個々のバンドルの中では、一組のオプションを使うようにしなければなりません。

例えば、IE9 以下のブラウザに対して CSS ファイルを条件的にインクルードするために、次のオプションを使うことが出来ます。

```php
public $cssOptions = ['condition' => 'lte IE9'];
```

こうすると、バンドルの中の CSS ファイルは下記の HTML タグを使ってインクルードされるようになります。

```html
<!--[if lte IE9]>
<link rel="stylesheet" href="path/to/foo.css">
<![endif]-->
```

生成された CSS のリンクタグを `<noscript>` の中に包むためには、次のように `cssOptions` を構成することが出来ます。

```php
public $cssOptions = ['noscript' => true];
```

JavaScript ファイルをページの head セクションにインクルードするためには、次のオプションを使います
(デフォルトでは、JavaScript ファイルは body セクションの最後にインクルードされます)。

```php
public $jsOptions = ['position' => \yii\web\View::POS_HEAD];
```

デフォルトでは、アセット・バンドルが発行されるときは、[[yii\web\AssetBundle::sourcePath]]
で指定されたディレクトリの中にある全てのコンテントが発行されます。
[[yii\web\AssetBundle::publishOptions|publishOptions]] プロパティを構成することによって、この振る舞いをカスタマイズすることが出来ます。
例えば、[[yii\web\AssetBundle::sourcePath]] の一個または数個のサブ・ディレクトリだけを発行するために、アセット・バンドル・クラスの中で下記のようにすることが出来ます。

```php
<?php
namespace app\assets;

use yii\web\AssetBundle;

class FontAwesomeAsset extends AssetBundle 
{
    public $sourcePath = '@bower/font-awesome'; 
    public $css = [ 
        'css/font-awesome.min.css', 
    ]; 
    public $publishOptions = [
        'only' => [
            'fonts/*',
            'css/*',
        ]
    ];
}  
```

上記の例は、["fontawesome" パッケージ](https://fontawesome.com/) のためのアセット・バンドルを定義するものです。
発行オプション `only` を指定して、`fonts` と `css` サブ・ディレクトリだけが発行されるようにしています。


### Bower と NPM のアセットのインストール <span id="bower-npm-assets"></span>

ほとんどの JavaScript/CSS パッケージは、[Bower](https://bower.io/) および/または [NPM](https://www.npmjs.com/) によって管理されています。
PHP の世界には PHP の依存を管理する Composer がありますが、PHP のパッケージと全く同じように
`composer.json` を使って Bower のパッケージも NPM のパッケージもロードすることが可能です。

このことを達成するために Composer の構成を少し修正しなければなりません。二つの方法があります。

___

#### asset-packagist レポジトリを使う

この方法は NPM または Bower のパッケージを必要とするプロジェクトの大半の要求を満たすことが出来ます。

> Note: 2.0.13 以降、ベーシック・アプリケーション・テンプレートとアドバンスト・アプリケーション・テンプレートはともに、
  デフォルトで asset-packagist を使うように前もって構成されていますので、このセクションは読み飛ばすことが出来ます。

プロジェクトの `composer.json` に、下記を追加します。

```json
"repositories": [
    {
        "type": "composer",
        "url": "https://asset-packagist.org"
    }
]
```

[アプリケーション構成情報](concept-configurations.md) の `@npm` と `@bower` の [エイリアス](concept-aliases.md) を修正します。

```php
$config = [
    ...
    'aliases' => [
        '@bower' => '@vendor/bower-asset',
        '@npm'   => '@vendor/npm-asset',
    ],
    ...
];
```

asset-packagist がどのようにして動作するのかは、[asset-packagist.org のサイト](https://asset-packagist.org) に説明があります。

#### fxp/composer-asset-plugin を使う

asset-packagist と異なって、composer-asset-plugin はアプリケーション構成の変更を少しも要求しません。
その代りに、次のコマンドを実行して特別な Composer プラグインをグローバルにインストールすることが要求されます。

```bash
composer global require "fxp/composer-asset-plugin:^1.4.1"
```

このコマンドによって [composer asset plugin](https://github.com/fxpio/composer-asset-plugin) がグローバルにインストールされ、
Bower と NPM パッケージの依存関係を Composer によって管理することが出来るようになります。
プラグインがインストールされた後は、あなたのコンピュータ上の全てのプロジェクトが `composer.json` を通じて Bower と NPM パッケージをサポートすることが出来ます。

Yii を使ってこれらのアセットを発行したい場合は、プロジェクトの `composer.json` に下記を追加して、
インストールされるパッケージが配置されるディレクトリを調整します。

```json
"config": {
    "fxp-asset": {
        "installer-paths": {
            "npm-asset-library": "vendor/npm",
            "bower-asset-library": "vendor/bower"
        }
    }
}
```

> Note: `fxp/composer-asset-plugin` は、asset-packagist に比べて、`composer update`
  コマンドを著しく遅くします。
 
____
 
Composer で Bower と NPM をサポートできるように構成した後は:

1. アプリケーションまたはエクステンションの `composer.json` ファイルを修正して、パッケージを `require` のエントリに入れます。
   ライブラリを参照するのに、`bower-asset/PackageName` (Bower パッケージ) または `npm-asset/PackageName` (NPM パッケージ)
   を使わなければなりません。
2. `composer update` を実行します。
3. アセット・バンドル・クラスを作成して、アプリケーションまたはエクステンションで使う予定の JavaScript/CSS ファイルをリストに挙げます。
   [[yii\web\AssetBundle::sourcePath|sourcePath]] プロパティは、`@bower/PackageName` または `@npm/PackageName` としなければなりません。
   これは、Composer が Bower または NPM パッケージを、このエイリアスに対応するディレクトリにインストールするためです。

> Note: パッケージの中には、全ての配布ファイルをサブ・ディレクトリに置くものがあります。
  その場合には、そのサブ・ディレクトリを [[yii\web\AssetBundle::sourcePath|sourcePath]] の値として指定しなければなりません。
  例えば、[[yii\web\JqueryAsset]] は `@bower/jquery` ではなく `@bower/jquery/dist` を使います。


## アセット・バンドルを使う <span id="using-asset-bundles"></span>

アセット・バンドルを使うためには、[[yii\web\AssetBundle::register()]] メソッドを呼んでアセット・バンドルを [ビュー](structure-views.md) に登録します。
例えば、次のようにしてビュー・テンプレートの中でアセット・バンドルを登録することが出来ます。

```php
use app\assets\AppAsset;
AppAsset::register($this);  // $this はビュー・オブジェクトを表す
```

> Info: [[yii\web\AssetBundle::register()]] メソッドは、[[yii\web\AssetBundle::basePath|basePath]] や [[yii\web\AssetBundle::baseUrl|baseUrl]] など、
  発行されたアセットに関する情報を含むアセット・バンドル・オブジェクトを返します。

他の場所でアセット・バンドルを登録しようとするときは、必要とされるビュー・オブジェクトを提供しなければなりません。
例えば、[ウィジェット](structure-widgets.md)・クラスの中でアセット・バンドルを登録するためには、`$this->view` によってビュー・オブジェクトを取得することが出来ます。

アセット・バンドルがビューに登録されるとき、舞台裏では、依存している全てのアセット・バンドルが Yii によって登録されます。
そして、アセット・バンドルがウェブからはアクセス出来ないディレクトリに配置されている場合は、
アセット・バンドルがウェブ・ディレクトリに発行されます。
その後、ビューがページをレンダリングするときに、登録されたバンドルのリストに挙げられている CSS と JavaScript ファイルのための `<link>` タグと `<script>` タグが生成されます。
これらのタグの順序は、登録されたバンドル間の依存関係、および、[[yii\web\AssetBundle::css]] と
[[yii\web\AssetBundle::js]] のプロパティのリストに挙げられたアセットの順序によって決定されます。


### 動的なアセット・バンドル <span id="dynamic-asset-bundles"></span>

アセット・バンドルは、通常の PHP クラスですので、内部のパラメータを動的に調整する追加のロジックを持つことが出来ます。
例えば、洗練された JavaScript ライブラリには、国際化の機能を、サポートする言語ごとに独立したソース・ファイルにパッケージして提供しているものもあります。
その場合、ライブラリの翻訳を動作させるためには、特定の '.js' ファイルをページに追加しなければなりません。
このことを [[yii\web\AssetBundle::init()]] メソッドをオーバーライドすることによって実現することが出来ます。

```php
namespace app\assets;

use yii\web\AssetBundle;
use Yii;

class SophisticatedAssetBundle extends AssetBundle
{
    public $sourcePath = '/path/to/sophisticated/src';
    public $js = [
        'sophisticated.js' // 常に使用されるファイル
    ];

    public function init()
    {
        parent::init();
        $this->js[] = 'i18n/' . Yii::$app->language . '.js'; // 動的に追加されるファイル
    }
}
```

個々のアセット・バンドルは、 [[yii\web\AssetBundle::register()]] によって返されるインスタンスによって調整することも出来ます。
例えば、

```php
use app\assets\SophisticatedAssetBundle;
use Yii;

$bundle = SophisticatedAssetBundle::register(Yii::$app->view);
$bundle->js[] = 'i18n/' . Yii::$app->language . '.js'; // 動的に追加されるファイル
```

> Note: アセット・バンドルの動的な調整はサポートされてはいますが、**推奨はされません**。
  予期しない副作用を引き起こしやすいので、可能であれば避けるべきです。


### アセット・バンドルをカスタマイズする <span id="customizing-asset-bundles"></span>

Yii は、[[yii\web\AssetManager]] によって実装されている `assetManager` という名前のアプリケーション・コンポーネントを通じてアセット・バンドルを管理します。
[[yii\web\AssetManager::bundles]] プロパティを構成することによって、アセット・バンドルの振る舞いをカスタマイズすることが出来ます。
例えば、デフォルトの [[yii\web\JqueryAsset]] アセット・バンドルはインストールされた jQuery の Bower パッケージにある `jquery.js` ファイルを使用します。
あなたは、可用性とパフォーマンスを向上させるために、Google によってホストされたバージョンを使いたいと思うかも知れません。
次のように、アプリケーションの構成情報で `assetManager` を構成することによって、それが達成できます。

```php
return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => [
                    'sourcePath' => null,   // バンドルを発行しない
                    'js' => [
                        '//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js',
                    ]
                ],
            ],
        ],
    ],
];
```

複数のアセット・バンドルも同様に [[yii\web\AssetManager::bundles]] によって構成することが出来ます。
配列のキーは、アセット・バンドルのクラス名 (最初のバック・スラッシュを除く) とし、
配列の値は、対応する [構成情報配列](concept-configurations.md) とします。

> Tip: アセット・バンドルの中で使うアセットを条件的に選択することが出来ます。
> 次の例は、開発環境では `jquery.js` を使い、そうでなければ `jquery.min.js` を使う方法を示すものです。
>
> ```php
> 'yii\web\JqueryAsset' => [
>     'js' => [
>         YII_ENV_DEV ? 'jquery.js' : 'jquery.min.js'
>     ]
> ],
> ```

無効にしたいアセット・バンドルの名前に `false` を結びつけることによって、一つまたは複数のアセット・バンドルを無効にすることが出来ます。
無効にされたアセット・バンドルをビューに登録した場合は、依存するバンドルは一つも登録されません。
また、ビューがページをレンダリングするときも、バンドルの中のアセットは一つもインクルードされません。
例えば、[[yii\web\JqueryAsset]] を無効化するために、次の構成情報を使用することが出来ます。

```php
return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => false,
            ],
        ],
    ],
];
```

[[yii\web\AssetManager::bundles]] を `false` にセットすることによって、*全て* のバンドルを無効にすることも出来ます。

[[yii\web\AssetManager::bundles]] によってなされたカスタマイズはアセット・バンドルの生成時、すなわち、オブジェクトのコンストラクタの段階で適用される、
ということを心に留めてください。
従って、[[yii\web\AssetManager::bundles]] のレベルで設定されたマッピングは、それ以後にバンドルのオブジェクトに対してなされる修正によって上書きされます。
具体的に言えば、[[yii\web\AssetBundle::init()]] メソッドの中での修正や、登録されたバンドル・オブジェクトに対する修正は、`AssetManager` の構成よりも優先されます。
以下に、[[yii\web\AssetManager::bundles]] によって設定されたマッピングが効力を持たない例を示します。

```php
// プログラムのソース・コード

namespace app\assets;

use yii\web\AssetBundle;
use Yii;

class LanguageAssetBundle extends AssetBundle
{
    // ...

    public function init()
    {
        parent::init();
        $this->baseUrl = '@web/i18n/' . Yii::$app->language; // AssetManager` では処理出来ない!
    }
}
// ...

$bundle = \app\assets\LargeFileAssetBundle::register(Yii::$app->view);
$bundle->baseUrl = YII_DEBUG ? '@web/large-files': '@web/large-files/minified'; // AssetManager` では処理出来ない!


// アプリケーション構成

return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
                'app\assets\LanguageAssetBundle' => [
                    'baseUrl' => 'http://some.cdn.com/files/i18n/en' // 効力を持たない!
                ],
                'app\assets\LargeFileAssetBundle' => [
                    'baseUrl' => 'http://some.cdn.com/files/large-files' // 効力を持たない!
                ],
            ],
        ],
    ],
];
```


### アセット・マッピング <span id="asset-mapping"></span>

時として、複数のアセット・バンドルで使われている 正しくない/互換でない アセット・ファイル・パスを「修正」したい場合があります。
例えば、バンドル A がバージョン 1.11.1 の `jquery.min.js` を使い、バンドル B がバージョン 2.1.1 の `jquery.js` を使っているような場合です。
それぞれのバンドルをカスタマイズすることで問題を修正することも出来ますが、それよりも簡単な方法は、*アセット・マップ* 機能を使って、正しくないアセットを望ましいアセットに割り付けることです。
そうするためには、以下のように [[yii\web\AssetManager::assetMap]] プロパティを構成します。

```php
return [
    // ...
    'components' => [
        'assetManager' => [
            'assetMap' => [
                'jquery.js' => '//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js',
            ],
        ],
    ],
];
```

[[yii\web\AssetManager::assetMap|assetMap]] のキーは修正したいアセットの名前であり、値は望ましいアセットのパスです。
アセット・バンドルをビューに登録するとき、[[yii\web\AssetBundle::css|css]] と
[[yii\web\AssetBundle::js|js]] の配列に含まれるすべてのアセット・ファイルの相対パスがこのマップと突き合わせて調べられます。
キーのどれかがアセット・ファイルのパス (利用できる場合は、[[yii\web\AssetBundle::sourcePath]] が前置されます)
の最後の部分と一致した場合は、対応する値によってアセットが置き換えられ、ビューに登録されます。
例えば、`my/path/to/jquery.js` というアセット・ファイルは `jquery.js` というキーにマッチします。

> Note: 相対パスを使って指定されたアセットだけがアセット・マッピングの対象になります。
  そして、置き換える側のアセットのパスは、絶対 URL であるか、[[yii\web\AssetManager::basePath]] からの相対パスであるかの、どちらかでなければなりません。


### アセット発行 <span id="asset-publishing"></span>

既に述べたように、アセット・バンドルがウェブからアクセス出来ないディレクトリに配置されている場合は、
バンドルがビューに登録されるときに、アセットがウェブ・ディレクトリにコピーされます。
このプロセスは *アセット発行* と呼ばれ、[[yii\web\AssetManager|アセット・マネージャ]] によって自動的に実行されます。

デフォルトでは、アセットが発行されるディレクトリは `@webroot/assets` であり、`@web/assets` という URL に対応するものです。
この場所は、[[yii\web\AssetManager::basePath|basePath]] と [[yii\web\AssetManager::baseUrl|baseUrl]]
のプロパティを構成してカスタマイズすることが出来ます。

ファイルをコピーすることでアセットを発行する代りに、OS とウェブ・サーバが許容するなら、シンボリック・リンクを使うことを考慮しても良いでしょう。
この機能は [[yii\web\AssetManager::linkAssets|linkAssets]] を `true` にセットすることで有効にすることが出来ます。

```php
return [
    // ...
    'components' => [
        'assetManager' => [
            'linkAssets' => true,
        ],
    ],
];
```

上記の構成によって、アセット・マネージャはアセット・バンドルを発行するときにソース・パスへのシンボリック・リンクを作成するようになります。
この方がファイルのコピーより速く、また、
発行されたアセットが常に最新であることを保証することも出来ます。


### キャッシュの廃棄 <span id="cache-busting"></span>

運用モードで動作しているウェブ・アプリケーションでは、アセットなどの静的なリソースに対する HTTP キャッシュを有効にするのが通例です。
この慣行の難点は、アセットを修正して運用サーバに配備したときに、ユーザのクライアントが HTTP キャッシュのせいで古いバージョンを使い続けるおそれが常にあるという点です。
この難点を克服するために、キャッシュ廃棄機能を使うことが出来ます。
これはバージョン 2.0.3 で導入された機能で、[[yii\web\AssetManager]]　を下記のように構成することで有効になります。
  
```php
return [
    // ...
    'components' => [
        'assetManager' => [
            'appendTimestamp' => true,
        ],
    ],
];
```

このようにすると、全ての発行されたアセットの URL の末尾に最終更新日時のタイムスタンプが追加されます。
例えば、`yii.js` に対する URL は `/assets/5515a87c/yii.js?v=1423448645"` のようなものになります。
パラメータ `v` が `yii.js` ファイルの最終更新日時のタイムスタンプを表しています。
これで、あなたがアセットを修正したときには、その URL も変更され、クライアントに最新バージョンのアセットを強制的に取得させることが出来ます。


## よく使われるアセット・バンドル <span id="common-asset-bundles"></span>

コアの Yii コードは多くのアセット・バンドルを定義しています。
その中で、下記のバンドルはよく使われるものであり、あなたのアプリケーションやエクステンションのコードでも参照することが出来るものです。

- [[yii\web\YiiAsset]]: 主として `yii.js` ファイルをインクルードするためのバンドルです。  このファイルはモジュール化された JavaScript のコードを編成するメカニズムを実装しています。
  また、`data-method` と `data-confirm` の属性に対する特別なサポートや、その他の有用な機能を提供します。
  `yii.js` に関する詳細な情報は [クライアント・スクリプトのセクション](output-client-scripts.md#yii.js) にあります。
- [[yii\web\JqueryAsset]]: jQuery の bower パッケージから `jquery.js` ファイルをインクルードします。
- [[yii\bootstrap\BootstrapAsset]]: Twitter Bootstrap フレームワークから CSS ファイルをインクルードします。
- [[yii\bootstrap\BootstrapPluginAsset]]: Bootstrap JavaScript プラグインをサポートするために、
  Twitter Bootstrap フレームワークから JavaScript ファイルをインクルードします。
- [[yii\jui\JuiAsset]]: jQuery UI ライブラリから CSS と JavaScript のファイルをインクルードします。

あなたのコードが、jQuery や jQuery UI または Bootstrap に依存する場合は、自分自身のバージョンを作るのではなく、これらの定義済みのアセット・バンドルを使用すべきです。
これらのバンドルのデフォルトの設定があなたの必要を満たさない時は、[アセット・バンドルをカスタマイズする](#customizing-asset-bundles)
の項で説明したように、それをカスタマイズすることが出来ます。


## アセット変換 <span id="asset-conversion"></span>

直接に CSS および/または JavaScript のコードを書く代りに、何らかの拡張構文を使って書いたものを特別なツールを使って CSS/JavaScript に変換する、ということを開発者はしばしば行います。
例えば、CSS コードのためには、[LESS](https://lesscss.org/) や [SCSS](https://sass-lang.com/) を使うことが出来ます。
また、JavaScript のためには、[TypeScript](https://www.typescriptlang.org/) を使うことが出来ます。

拡張構文を使ったアセット・ファイルをアセット・バンドルの中の [[yii\web\AssetBundle::css|css]] と [[yii\web\AssetBundle::js|js]] のリストに挙げることが出来ます。例えば、

```php
class AppAsset extends AssetBundle
{
    public $basePath = '@webroot';
    public $baseUrl = '@web';
    public $css = [
        'css/site.less',
    ];
    public $js = [
        'js/site.ts',
    ];
    public $depends = [
        'yii\web\YiiAsset',
        'yii\bootstrap\BootstrapAsset',
    ];
}
```

このようなアセット・バンドルをビューに登録すると、[[yii\web\AssetManager|アセット・マネージャ]] が自動的にプリ・プロセッサ・ツールを走らせて、
認識できた拡張構文のアセットを CSS/JavaScript に変換します。
最終的にビューがページをレンダリングするときには、ビューは元の拡張構文のアセットではなく、
変換後の CSS/JavaScript ファイルをページにインクルードします。

Yii はファイル名の拡張子を使って、アセットが使っている拡張構文を識別します。
デフォルトでは、下記の構文とファイル名拡張子を認識します。

- [LESS](https://lesscss.org/): `.less`
- [SCSS](https://sass-lang.com/): `.scss`
- [Stylus](https://stylus-lang.com/): `.styl`
- [CoffeeScript](https://coffeescript.org/): `.coffee`
- [TypeScript](https://www.typescriptlang.org/): `.ts`

Yii はインストールされたプリ・プロセッサ・ツールに頼ってアセットを変換します。
例えば、[LESS](https://lesscss.org/) を使うためには、`lessc` プリ・プロセッサ・コマンドをインストールしなければなりません。

下記のように [[yii\web\AssetManager::converter]] を構成することで、
プリ・プロセッサ・コマンドとサポートされる拡張構文をカスタマイズすることが出来ます。

```php
return [
    'components' => [
        'assetManager' => [
            'converter' => [
                'class' => 'yii\web\AssetConverter',
                'commands' => [
                    'less' => ['css', 'lessc {from} {to} --no-color'],
                    'ts' => ['js', 'tsc --out {to} {from}'],
                ],
            ],
        ],
    ],
];
```

上記においては、サポートされる拡張構文が [[yii\web\AssetConverter::commands]] プロパティによって定義されています。
配列のキーはファイルの拡張子 (先頭のドットは省く) であり、
配列の値は結果として作られるアセット・ファイルの拡張子とアセット変換を実行するためのコマンドです。
コマンドの中の `{from}` と `{to}` のトークンは、ソースのアセット・ファイルのパスとターゲットのアセット・ファイルのパスに置き換えられます。

> Info: 上記で説明した方法の他にも、拡張構文のアセットを扱う方法はあります。
  例えば、[grunt](http://gruntjs.com/) のようなビルド・ツールを使って、拡張構文のアセットをモニターし、
  自動的に変換することが出来ます。
  この場合は、元のファイルではなく、結果として作られる CSS/JavaScript ファイルをアセット・バンドルのリストに挙げなければなりません。


## アセットを結合して圧縮する <span id="combining-compressing-assets"></span>

ウェブ・ページは数多くの CSS および/または JavaScript ファイルをインクルードすることがあり得ます。
HTTP リクエストの数とこれらのファイルの全体としてのダウンロード・サイズを削減するためによく用いられる方法は、複数の CSS/JavaScript ファイルを結合して圧縮し、一つまたはごく少数のファイルにまとめることです。
そして、ウェブ・ページでは元のファイルをインクルードする代りに、圧縮されたファイルをインクルードする訳です。
 
> Info: アセットの結合と圧縮は、通常はアプリケーションが本番モードにある場合に必要になります。
  開発モードにおいては、たいていは元の CSS/JavaScript ファイルを使う方がデバッグのために好都合です。

次に、既存のアプリケーション・コードを修正する必要なしに、
アセット・ファイルを結合して圧縮する方法を紹介します。

1. アプリケーションの中で、結合して圧縮する予定のアセット・バンドルを全て探し出す。
2. これらのバンドルを一個か数個のグループにまとめる。どのバンドルも一つのグループにしか属することが出来ないことに注意。
3. 各グループの CSS ファイルを一つのファイルに結合/圧縮する。JavaScript ファイルに対しても同様にこれを行う。
4. 各グループに対して新しいアセット・バンドルを定義する。
   * [[yii\web\AssetBundle::css|css]] と [[yii\web\AssetBundle::js|js]] のプロパティに、
     それぞれ、結合された CSS ファイルと JavaScript ファイルをセットする。
   * 各グループに属する元のアセット・バンドルをカスタマイズして、[[yii\web\AssetBundle::css|css]] と
     [[yii\web\AssetBundle::js|js]] のプロパティを空にし、[[yii\web\AssetBundle::depends|depends]]
     プロパティにグループのために作られた新しいバンドルを指定する。

この方法を使うと、ビューでアセット・バンドルを登録したときに、
元のバンドルが属するグループのための新しいアセット・バンドルが自動的に登録されるようになります。
そして、結果として、結合/圧縮されたアセット・ファイルが、元のファイルの代りに、ページにインクルードされます。


### 一例 <span id="example"></span>

上記の方法をさらに説明するために一つの例を挙げましょう。

あなたのアプリケーションが二つのページ、X と Y を持つと仮定します。ページ X はアセット・バンドル A、B、C を使用し、ページ Y はアセット・バンドル B、C、D を使用します。

これらのアセット・バンドルを分割する方法は二つあります。一つは単一のグループを使用して全てのアセット・バンドルを含める方法です。
もう一つは、A をグループ X に入れ、D をグループ Y に入れ、そして、B と C をグループ S に入れる方法です。
どちらが良いでしょう? 場合によります。
最初の方法の利点は、二つのページが同一の結合された CSS と JavaScript のファイルを共有するため、HTTP キャッシュの効果が高くなることです。
その一方で、単一のグループが全てのバンドルを含むために、結合された CSS と JavaScript のファイルはより大きくなり、従って最初のファイル転送時間はより長くなります。
この例では話を簡単にするために、最初の方法、すなわち、全てのバンドルを含む単一のグループを使用することにします。

> Info: アセット・バンドルをグループに分けることは些細な仕事ではありません。通常、そのためには、いろいろなページのさまざまなアセットの現実世界での転送量を分析することが必要になります。
  とりあえず、最初は、簡単にするために、単一のグループから始めて良いでしょう。

既存のツール (例えば [Closure Compiler](https://developers.google.com/closure/compiler/) や [YUI Compressor](https://github.com/yui/yuicompressor/)) を使って、
全てのバンドルにある CSS と JavaScript のファイルを結合して圧縮します。
ファイルは、バンドル間の依存関係を満たす順序に従って結合しなければならないことに注意してください。
例えば、バンドル A が B に依存し、B が C と D の両方に依存している場合は、アセット・ファイルの結合順は、
最初に C と D、次に B、そして最後に A としなければなりません。

結合と圧縮が完了すると、一つの CSS ファイルと一つの JavaScript ファイルが得られます。
それらは、`all-xyz.css` および `all-xyz.js` と命名されたとしましょう。
ここで `xyz` は、ファイル名をユニークにして HTTP キャッシュの問題を避けるために使用されるタイムスタンプまたはハッシュを表します。

いよいよ最終ステップです。
アプリケーションの構成情報の中で、[[yii\web\AssetManager|アセット・マネージャ]] を次のように構成します。

```php
return [
    'components' => [
        'assetManager' => [
            'bundles' => [
                'all' => [
                    'class' => 'yii\web\AssetBundle',
                    'basePath' => '@webroot/assets',
                    'baseUrl' => '@web/assets',
                    'css' => ['all-xyz.css'],
                    'js' => ['all-xyz.js'],
                ],
                'A' => ['css' => [], 'js' => [], 'depends' => ['all']],
                'B' => ['css' => [], 'js' => [], 'depends' => ['all']],
                'C' => ['css' => [], 'js' => [], 'depends' => ['all']],
                'D' => ['css' => [], 'js' => [], 'depends' => ['all']],
            ],
        ],
    ],
];
```

[アセット・バンドルをカスタマイズする](#customizing-asset-bundles) の項で説明したように、上記の構成によって元のバンドルは全てデフォルトの振る舞いを変更されます。
具体的にいえば、バンドル A、B、C、D は、もはやアセット・ファイルを一つも持っていません。
この4つは、それぞれ、結合された `all-xyz.css` と `all-xyz.js` ファイルを持つ `all` バンドルに依存するようになりました。
結果として、ページ X では、バンドル A、B、C から元のソース・ファイルをインクルードする代りに、これら二つの結合されたファイルだけがインクルードされます。
同じことはページ Y でも起ります。

最後にもう一つ、上記の方法をさらにスムーズに運用するためのトリックがあります。
アプリケーションの構成情報ファイルを直接修正する代りに、バンドルのカスタマイズ用の配列を独立したファイルに置いて、条件によってそのファイルをアプリケーションの構成情報にインクルードすることが出来ます。
例えば、

```php
return [
    'components' => [
        'assetManager' => [
            'bundles' => require __DIR__ . '/' . (YII_ENV_PROD ? 'assets-prod.php' : 'assets-dev.php'),  
        ],
    ],
];
```

つまり、アセット・バンドルの構成情報配列は、本番モードのものは `assets-prod.php` に保存し、
開発モードのものは `assets-dev.php` に保存するという訳です。

> Note: このアセット結合のメカニズムは、登録されるアセット・バンドルのプロパティをオーバーライドできるという [[yii\web\AssetManager::bundles]] の機能に基づいています。
  しかし、既に上で述べたように、この機能は、[[yii\web\AssetBundle::init()]]
  メソッドの中やバンドルが登録された後で実行されるアセット・バンドルの修正をカバーしていません。
  そのような動的なバンドルの使用は、アセット結合をする際には避けなければなりません。


### `asset` コマンドを使う <span id="using-asset-command"></span>

Yii は、たった今説明した方法を自動化するための `asset` という名前のコンソール・コマンドを提供しています。

このコマンドを使うためには、最初に構成情報ファイルを作成して、どのアセット・バンドルが結合されるか、
そして、それらがどのようにグループ化されるかを記述しなければなりません。
`asset/template` サブ・コマンドを使って最初にテンプレートを生成し、それをあなたの要求に合うように修正することが出来ます。

```
yii asset/template assets.php
```

上記のコマンドは、カレント・ディレクトリに `assets.php` というファイルを生成します。このファイルの内容は以下のようなものです。

```php
<?php
/**
 * "yii asset" コンソール・コマンドのための構成情報ファイル
 * コンソール環境では、'@webroot' や '@web' のように、存在しないパス・エイリアスがあり得ることに注意してください。
 * これらの欠落したパス・エイリアスは手作業で定義してください。
 */
return [
    // JavaScript ファイルの圧縮のためのコマンド/コールバックを調整。
    'jsCompressor' => 'java -jar compiler.jar --js {from} --js_output_file {to}',
    // CSS ファイルの圧縮のためのコマンド/コールバックを調整。
    'cssCompressor' => 'java -jar yuicompressor.jar --type css {from} -o {to}',
    // 圧縮後にアセットのソースを削除するかどうか。
    'deleteSource' => false,
    // 圧縮するアセット・バンドルのリスト。
    'bundles' => [
        // 'yii\web\YiiAsset',
        // 'yii\web\JqueryAsset',
    ],
    // 圧縮出力用のアセット・バンドル。
    'targets' => [
        'all' => [
            'class' => 'yii\web\AssetBundle',
            'basePath' => '@webroot/assets',
            'baseUrl' => '@web/assets',
            'js' => 'js/all-{hash}.js',
            'css' => 'css/all-{hash}.css',
        ],
    ],
    // アセット・マネージャの構成情報
    'assetManager' => [
    ],
];
```

このファイルを修正して、どのバンドルを結合するつもりであるかを `bundles` オプションで指定しなければなりません。
`targets` オプションでは、バンドルがどのようにグループに分割されるかを指定しなければなりません。
既に述べたように、一つまたは複数のグループを定義することが出来ます。

> Note: パス・エイリアス `@webroot` および `@web` はコンソール・アプリケーションでは利用できませんので、
  これらは構成情報の中で明示的に定義しなければなりません。

JavaScript ファイルは結合され、圧縮されて `js/all-{hash}.js` に保存されます。ここで {hash} は、
結果として作られたファイルのハッシュで置き換えられるものです。

`jsCompressor` と `cssCompressor` のオプションは、JavaScript と CSS の結合/圧縮を実行するコンソール・コマンドまたは PHP コールバックを指定するものです。
デフォルトでは、Yii は JavaScript ファイルの結合に [Closure Compiler](https://developers.google.com/closure/compiler/) を使い、
CSS ファイルの結合に [YUI Compressor](https://github.com/yui/yuicompressor/) を使用します。
あなたの好みのツールを使うためには、手作業でツールをインストールしたり、オプションを修正したりしなければなりません。


この構成情報ファイルを使い、`asset` コマンドを走らせて、アセット・ファイルを結合して圧縮し、
同時に、新しいアセット・バンドルの構成情報ファイル `assets-prod.php` を生成することが出来ます。
 
```
yii asset assets.php config/assets-prod.php
```

直前の項で説明したように、
この生成された構成情報ファイルをアプリケーションの構成情報にインクルードすることが出来ます。

> Note: アプリケーションのアセット・バンドルを [[yii\web\AssetManager::bundles]] または [[yii\web\AssetManager::assetMap]] によってカスタマイズしており、
そのカスタマイズを圧縮のソース・ファイルにも適用したい場合は、それらのオプションを `asset` コマンドの構成ファイルの
`assetManager` のセクションに含めなければいけません。

> Note: 圧縮のソースを指定する場合は、パラメータが動的に (例えば `init()` メソッドの中や登録後に) 修正されるアセット・バンドルを避けなければなりません。
  なぜなら、パラメータの動的な修正は、圧縮後は正しく働かない可能性があるからです。


> Info: `asset` コマンドを使うことは、アセットの結合・圧縮のプロセスを自動化する唯一の選択肢ではありません。
  優秀なタスク実行ツールである [grunt](http://gruntjs.com/) を使っても、同じ目的を達することが出来ます。


### アセット・バンドルをグループ化する <span id="grouping-asset-bundles"></span>

直前の項において、全てのアセット・バンドルを一つに結合して、アプリケーションで参照されるアセット・ファイルに対する HTTP リクエストを最小化する方法を説明しました。
現実には、それが常に望ましいわけではありません。
例えば、あなたのアプリケーションがフロントエンドとバックエンドを持っており、
それぞれが異なる一群の JavaScript と CSS ファイルを使う場合を想像してください。
この場合、両方の側の全てのアセット・バンドルを一つに結合するのは合理的ではありません。
何故なら、フロントエンドのためのアセット・バンドルはバックエンドでは使用されませんから、フロントエンドのページがリクエストされているときにバックエンドのアセットを送信するのはネットワーク帯域の浪費です。

上記の問題を解決するために、アセット・バンドルをグループ化して、グループごとにアセット・バンドルを結合することが出来ます。
下記の構成情報は、アセット・バンドルをグループ化する方法を示すものです。

```php
return [
    ...
    // 出力されるバンドルをグループ化する
    'targets' => [
        'allShared' => [
            'js' => 'js/all-shared-{hash}.js',
            'css' => 'css/all-shared-{hash}.css',
            'depends' => [
                // バックエンドとフロントエンドで共有される全てのアセットを含める
                'yii\web\YiiAsset',
                'app\assets\SharedAsset',
            ],
        ],
        'allBackEnd' => [
            'js' => 'js/all-{hash}.js',
            'css' => 'css/all-{hash}.css',
            'depends' => [
                // バックエンドだけのアセットを含める
                'app\assets\AdminAsset'
            ],
        ],
        'allFrontEnd' => [
            'js' => 'js/all-{hash}.js',
            'css' => 'css/all-{hash}.css',
            'depends' => [], // 残りの全てのアセットを含める
        ],
    ],
    ...
];
```

ご覧のように、アセット・バンドルは三つのグループ、すなわち、`allShared`、`allBackEnd` および `allFrontEnd` に分けられています。
そして、それぞれが適切な一群のアセット・バンドルに依存しています。例えば、`allBackEnd` は `app\assets\AdminAsset` に依存しています。
この構成情報によって `asset` コマンドを実行すると、上記の定義に従ってアセット・バンドルが結合されます。

> Info: ターゲット・バンドルのうちの一つについて `depends` の構成を空のままにしておくことが出来ます。
  そのようにすると、他のターゲット・バンドルが依存しないために残された全てのアセット・バンドルが、このターゲット・バンドルに含まれるようになります。
