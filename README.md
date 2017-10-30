# PHP B

本リポジトリは aws の CodeBuild を使用し、  
他リポジトリの内容を composer で取得しながらビルドをするサンプルです

[PHP A](https://github.com/opst-ezaki/php-a) と共同で使用することを前提としています。

## 操作手順

こちらでは PHP B の成果物を作成する手順までの解説を記載しています。

本内容は PHP A のコンテナが ECR 上に存在していることを前提としているため、 [PHP A](https://github.com/opst-ezaki/php-a) の手順を完了させていない場合は、先にそちらを行ってください。

**PHP B の内容のみでは実行が出来ません**

1. [ECS](https://ap-northeast-1.console.aws.amazon.com/ecs/home?region=ap-northeast-1#/repositories) のページを開き、PHP A のリポジトリへのアクセス許可を追加します
    - PHP A のリポジトリ名をクリック
    - `アクセス許可` のタブを開く
    - タブ内の `追加` をクリック
    - 以下、次のように入力
        - ステートメント識別子: `PHP A statement`
        - 効果: `許可`
        - プリンシパル
            - 全員: `オフ`
            - AWS アカウント番号または [service].amazonaws.com: `157144849617`
            - 全ての IAM エンティティ: (選択無し)
            - アクション: `プル専用アクション`
    - 以上を入力し、上に戻り `すべて保存` をクリック
    
    ※ AWS アカウント番号の `157144849617` は、東京リージョンにおける CodeBuild 内部アカウントの番号

1. [S3](https://s3.console.aws.amazon.com/s3/home?region=ap-northeast-1) のページを開き `バケットを作成する` をクリックする
    - 名前とリージョン
        - バケット名: `{yourname}-php-b`
        - リージョン: `アジアパシフィック(東京)`
        - 既存のバケットから設定をコピー: (省略)
    -  プロパティの設定
        - バージョニング: `有効`
        - サーバーアクセスのログ記録: `無効`
        - Tags: `0 Tags`
        - オブジェクトレベルのログ記録: `無効`
    - アクセス許可の設定: デフォルトのまま
    以上を設定し、 `バケットを作成` を選択

1. [CodeBuild](https://ap-northeast-1.console.aws.amazon.com/codebuild/home?region=ap-northeast-1#/projects) のページを開き、 `プロジェクトの作成` をクリックする
    - プロジェクトの設定
            - プロジェクト名: `php-b`
            - 説明: なし
        - ソース：ビルドの対象
            - プロバイダ: `Github`
            - リポジトリ: `パブリックのリポジトリの使用`
            - リポジトリの URL : `https://github.com/opst-ezaki/php-b`
            - Webhook: オフ
        - 環境：ビルド方法
            - 環境イメージ: `Docker イメージの指定`
            - 環境タイプ: `Linux`
            - カスタムイメージタイプ: `Amazon ECR`
            - Amazon ECR レポジトリ: `php-a`
            - Amazon ECR イメージ: `latest`
            - ビルド仕様: `ソースコードのルートディレクトリの buildspeck.yml` を使用
            - buildspeck 名: buildspec.yml (変更なし)
        - アーティファクト: 子のビルドプレジェクトからアーティファクトを配置する場所
            - タイプ: `Amazon S3`
            - 名前: (指定なし)
            - パス: (指定なし)
            - 名前空間のタイプ: `なし`
            - バケット名: `{yourname}-php-b`
        - サービスロール
            - `アカウントでサービスロールを作成`
            - ロール名: `codebuild-php-b-service-role`
        以上を設定し、 `続行` -> `保存` をクリック

1. [CodeBuild の php-b プロジェクト](https://ap-northeast-1.console.aws.amazon.com/codebuild/home?region=ap-northeast-1#/projects/php-b/view)へ移動し、 `ビルド履歴` から `ビルドの開始` -> `ビルドの開始` をクリック

1. ビルドが正常に終了し、S3 のバケットに `archive.zip` が保存されていることを確認する

1. `archive.zip` をダウンロードし解凍、 `php index.php` を実行し、 `Hello` が出力されることを確認 (本メッセージは PHP A の Foo Class に記述されているものを実行している)