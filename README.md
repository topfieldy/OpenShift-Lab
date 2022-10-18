# Technology Zoneを利用して、OpenShift環境にアプリケーションをデプロイ
Technology Zoneを利用して、OpenShift環境を作成し、公開されているアプリケーションをデプロイして実際に動かしてみました。実施してみて、難しかったところや詰まったところも含めて記載していきます。

## Technology ZoneにOpenShift環境を払出し
OpenShiftの環境のreserve方法を解説します。
1. [Technology Zone](https://techzone.ibm.com/)にアクセス
Technology Zoneにアクセスし、IBMidを利用してログインします。

2. 利用する環境を探して、手順に従って環境のreserveする
今回は[Red Hat OpenShift on IBM Cloud basics lab - Environment](https://techzone.ibm.com/collection/roks-basics-lab#tab-1)の環境を利用しています。
画面キャプチャの右側の環境です。「Reserve」をクリックし、「Reserce now」を選択し、「Submit」をクリックすると注文の詳細設定の画面に遷移します。
<img width="1237" alt="スクリーンショット 2022-10-17 10 18 16" src="https://user-images.githubusercontent.com/112134163/196212028-8fc8eff2-14d8-480d-acb7-bacc85bc1161.png">

3. 環境を作成する目的や詳細など必須項目を埋めて「Submit」
Purposeは下記から選択する必要があります。普段自分の学習用に使う場合は、Opportunity Codeが求められない「Practice/Self-Education」を選ぶことが多いです。その他、利用したいDCや利用開始日時等を入力します。
<img width="1065" alt="スクリーンショット 2022-10-17 10 21 27" src="https://user-images.githubusercontent.com/112134163/196212759-0ad01696-f6cc-40bf-8c93-5de787dd3a41.png">

4. 環境のプロビジョニング完了を待つ
約10分ほどで環境がプロビジョニングされ、メールで通知がきました。

5. OpenShift環境がプロビジョニングされていることを確認
完了メールか、Technology ZoneのMy Reservationから作成された環境を確認することができます。
<img width="1073" alt="スクリーンショット 2022-10-18 13 51 52" src="https://user-images.githubusercontent.com/112134163/196338566-d733b5bc-b53a-47bc-b3ac-9aa45620d39a.png">

## OpenShift上にGitHubに公開されているアプリケーションをデプロイ
実際にGitHubに公開されているアプリケーションをデプロイしていきます。監視用アプリケーションの[Instanaのアプリ](https://github.com/instana/robot-shop)を使用してデプロイします。
<img width="1440" alt="スクリーンショット 2022-10-18 0 14 14" src="https://user-images.githubusercontent.com/112134163/196215705-763f36ef-b356-4983-b1ae-3bb6bb53e464.png">

### 前提条件1　OpenShiftクラスターの設定
OpenShift上にデプロイをするので、[ガイド](https://github.com/instana/robot-shop/tree/master/OpenShift)に従い設定を行いました。
ここでは、作成したOpenShiftクラスターにCLIでログインをして、Administratorの権限で「robot-shop」というProjectを作成しました。OpenShiftではデフォルト状態で「restricted」というSCC(Security Context Constraints)というセキュリティ制約がかかっています。SCCとはPodのパーミッションを制御する機能です。ただ今回はモニタリングをするので、Root起動を許可しています。
![OCP 4 x](https://user-images.githubusercontent.com/112134163/196216692-09278479-d15c-4024-9fde-62627fe220ba.png)

#### 1行目ではOpenShiftクラスターにログインをすることになっていますが、下記の手順で別のコマンドを利用してログインしました。
1. OpenShiftクラスターのOpenShift Webコンソールにアクセス
2. Developerメニューのトポロジー
3. 自分のアカウント名の右にあるプルダウンから「ログインコマンドのコピー」（画面右上）
4. Display Token
5. Log in with this token のコマンドをコピー（OpenShift CLIをローカルにインストールしておく必要がある ※前提条件2）
6. CLIにコピーしたログインコマンドをペースト

#### Helmのインストール
Helmとは、リポジトリからのインストールや、Helmによってデプロイされたアプリの管理をCLIで簡易化するためのOSSです。Kubernetes向けのパッケージマネージャーとしてよく使われるようです。LinuxにおけるyumやRPM、MacOSにおけるHomebrewなどと同様のツールと考えると理解しやすいです。

### 前提条件2　OpenShift CLI(oc)のインストール
[IBM CloudのDocsのガイド](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift-cli&locale=ja)を参考にしました。

### 実際にデプロイをする

手順1. Developerメニューの「+追加」からGit リポジトリーの「Gitからのインポート」に移動
![スクリーンショット 2022-10-18 1 18 18](https://user-images.githubusercontent.com/112134163/196229876-b21b55b5-36e5-4694-ba4c-ae8e54385da2.png)

手順2. Git リポジトリーのURLを入力。今回は[Robot-shop](https://github.com/instana/robot-shop)で作成
![スクリーンショット 2022-10-18 1 19 57](https://user-images.githubusercontent.com/112134163/196230208-333c9ae9-b69c-40ab-80b8-e25413ea2963.png)

手順3. インポートストラテジーの編集で、今回最適なNode.jsで作成
![スクリーンショット 2022-10-18 1 22 01](https://user-images.githubusercontent.com/112134163/196230617-6c375c01-6439-4991-a8d6-914e309c1436.png)

手順4. デプロイされたアプリを確認
トポロジーからデプロイを確認できます。いくつもアプリケーションがデプロイされているのが確認できます。
<img width="1148" alt="スクリーンショット 2022-10-18 13 44 26" src="https://user-images.githubusercontent.com/112134163/196337451-52a1bbb0-0a5f-4cd5-9203-da998ca04af7.png">


## 実際にアプリを開いてみる。
しっかりと動いていることが確認できました。
<img width="1440" alt="スクリーンショット 2022-10-18 9 59 58" src="https://user-images.githubusercontent.com/112134163/196337662-a48ea325-da1c-4521-8c6c-6aac63a5d1fd.png">

#### アプリのデプロイの時間と難易度が低くなります
OpenShiftを利用したので短時間で簡単にアプリをデプロイできました。従来の手作業でやっていたアプリのデプロイと比較して、裏ではOpenShiftが自動でプロセスをやってくれるので、人が実施するプロセスの数や時間が大幅に短縮して、簡単に速くアプリのデプロイが可能になります。

今回のデプロイメントは、IBM社員用のアカウント上に作成しています。Technology Zoneで払い出したOpenShift環境では開発者の権限が少なく、アプリをデプロイすることができませんでした。
現在、環境の作成者に権限を追加できないか依頼しているところです。社員アカウントでは強い権限があるので、デプロイに必要な権限があり、無事に完了しました。

