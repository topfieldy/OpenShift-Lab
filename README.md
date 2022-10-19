# Technology Zoneを利用して、OpenShift環境にアプリケーションをデプロイ
Technology Zoneを利用して、OpenShift環境を作成し、公開されているアプリケーションをデプロイして実際に動かしてみました。実施してみて、難しかったところや詰まったところも含めて紹介していきます。

## Technology ZoneにOpenShift環境を払出し
OpenShiftの環境のreserve方法を解説します。
1. [Technology Zone](https://techzone.ibm.com/)にアクセス
Technology Zoneにアクセスし、IBMidを利用してログインします。

2. 利用する環境を探して、手順に従って環境のreserveする
今回は[Red Hat OpenShift on IBM Cloud basics lab - Environment](https://techzone.ibm.com/collection/roks-basics-lab#tab-1)の環境を利用しています。
画面キャプチャの右側の環境です。「Reserve」をクリックし、「Reserce now」を選択し、「Submit」をクリックすると注文の詳細設定の画面に遷移します。
<img width="1237" alt="スクリーンショット 2022-10-17 10 18 16" src="https://user-images.githubusercontent.com/112134163/196212028-8fc8eff2-14d8-480d-acb7-bacc85bc1161.png">

3. 環境を作成する目的や詳細など必須項目を埋めて「Submit」
Purposeは下記から選択する必要があります。普段自分の学習用に使う場合は、無条件で72時間まで使える「Practice/Self-Education」を選ぶことが多いです。その他、利用したいDCや利用開始日時等を入力します。
<img width="1065" alt="スクリーンショット 2022-10-17 10 21 27" src="https://user-images.githubusercontent.com/112134163/196212759-0ad01696-f6cc-40bf-8c93-5de787dd3a41.png">

4. 環境のプロビジョニング完了を待つ
約10分ほどで環境がプロビジョニングされ、メールで通知がきました。

5. OpenShift環境がプロビジョニングされていることを確認
完了メールか、Technology ZoneのMy Reservationから作成された環境を確認することができます。
<img width="1073" alt="スクリーンショット 2022-10-18 13 51 52" src="https://user-images.githubusercontent.com/112134163/196338566-d733b5bc-b53a-47bc-b3ac-9aa45620d39a.png">

## GitHubに公開されているアプリケーションをOpenShift上にデプロイ
実際にGitHubに公開されているアプリケーションをデプロイしていきます。監視用アプリケーションの[Instanaのアプリ Robot-shop](https://github.com/instana/robot-shop)を使用してデプロイします。
<img width="1440" alt="スクリーンショット 2022-10-18 0 14 14" src="https://user-images.githubusercontent.com/112134163/196215705-763f36ef-b356-4983-b1ae-3bb6bb53e464.png">

### 前提条件1 OpenShift CLI(oc)のインストール
[IBM CloudのDocsのガイド](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift-cli&locale=ja)を参考にしました。


### 前提条件2 Helmのインストール
Helmとは、リポジトリからのインストールや、Helmによってデプロイされたアプリの管理をCLIで簡易化するためのOSSです。Kubernetes向けのパッケージマネージャーとしてよく使われるようです。LinuxにおけるyumやRPM、MacOSにおけるHomebrewなどと同様のツールと考えると理解しやすいです。


## デプロイの手順
OpenShift上にデプロイをするので、[ガイド](https://github.com/instana/robot-shop/tree/master/OpenShift)に従い設定を行いました。
下記のコマンドを実行することで、アプリケーションをデプロイできました。
```
export KUBECONFIG=/path/to/oc/cluster/dir/auth/kubeconfig
oc adm new-project robot-shop
oc adm policy add-scc-to-user anyuid -z default -n robot-shop
oc adm policy add-scc-to-user privileged -z default -n robot-shop
cd robot-shop/K8s
helm install robot-shop --set openshift=true -n robot-shop helm
```



##### それぞれのコマンドの意味について解説していきます。
```
export KUBECONFIG=/path/to/oc/cluster/dir/auth/kubeconfig
```
ここでは、OpenShiftクラスターへログインをしています。私の場合は別の方法でログインをしました。
1. OpenShiftクラスターのOpenShift Webコンソールにアクセス
2. Developerメニューのトポロジー
3. 画面右上にある自分のアカウント名の右側にあるプルダウンから「ログインコマンドのコピー」
4. Display Tokenをクリック
5. Log in with this token のコマンドをコピー（※前提条件1が必須）
6. CLIにコピーしたログインコマンドをペースト  


```
oc adm new-project robot-shop
```
「oc」とはOpenShiftのコマンドで、「adm」はAdministratorの権限を使って、「robot-shop」という新しいProjectを作成しています。 
<br>
<br>

```
oc adm policy add-scc-to-user anyuid -z default -n robot-shop
oc adm policy add-scc-to-user privileged -z default -n robot-shop
```
OpenShiftではデフォルト状態で「restricted」というSCC(Security Context Constraints)のセキュリティ制約がかかっています。SCCとはPodのパーミッションを制御する機能です。
今回のアプリは制約が強いので、Projectに特別な強い権限を付与しています。
ただ、今回使用したOpenShift環境は権限の範囲が狭く、下記のようにErrorとなってしまい、Projectに強い権限を付与することができませんでした。
結果として、権限が足りなくて、この先のHelmでインストールするところでアプリのデプロイに失敗ということです。

```
oc adm　 policy add-scc-to-user anyuid -z default -n robot-shop
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io "system:openshift:scc:anyuid" is forbidden: User "IAM#yuki.uehara2@ibm.com" cannot get resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "robot-shop"

oc adm policy add-scc-to-user privileged -z default -n robot-shop
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io "system:openshift:scc:privileged" is forbidden: User "IAM#yuki.uehara2@ibm.com" cannot get resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "robot-shop"  
```
<br>

ここからは権限の範囲が広く設定してある別の[Technology ZoneのOpenShift環境](https://techzone.ibm.com/collection/fyre-ocp-clusters#)に切り替えてやっていきます。
やり方としては、ここまでやってきたことと同様に進めていきます。 
<br>
<br>

```
cd robot-shop/K8s
helm install robot-shop --set openshift=true -n robot-shop helm
```
cdコマンドで、Helmが入っている場所に移動し、その後に実際にアプリをデプロイするコマンドを実行しています。

※前提条件2 として事前にHelmをインストールしておかないとコマンドが実行できないので注意が必要です。 
<br>
<br>


### デプロイされたアプリを確認
トポロジーからデプロイを確認することができます。いくつもアプリケーションがデプロイされているのが確認できるので、アプリのデプロイ成功です。
<img width="1148" alt="スクリーンショット 2022-10-18 13 44 26" src="https://user-images.githubusercontent.com/112134163/196337451-52a1bbb0-0a5f-4cd5-9203-da998ca04af7.png"> 
<br>
<br>

## 実際にアプリを開いてみる
アプリのアクセス方法は、「トポロジー」のデプロイしたアプリの中から、「wed」を選び、「リソース」のタブを開きます。webのPodポートが「8080」だと確認できます。次にサービスの中の「web」のリンク先にアクセスします。
<img width="1078" alt="スクリーンショット 2022-10-18 21 38 55" src="https://user-images.githubusercontent.com/112134163/196431554-322e68da-9e9f-49d7-b037-9828934da5ea.png"> 

外部ロードーバランサーのIPが「163.73.69.51」と確認できる。
<img width="1070" alt="スクリーンショット 2022-10-18 21 42 38" src="https://user-images.githubusercontent.com/112134163/196432390-85425722-871d-4e0f-bd9b-fba3ab399ba4.png"> 

```
ブラウザにIPアドレスを入力

http://<外部ロードバランサーのIP>:<web Podポート>
http://163.73.69.51:8080
```
その結果しっかりと動いていることが確認できました。
<img width="1078" alt="スクリーンショット 2022-10-18 21 48 20" src="https://user-images.githubusercontent.com/112134163/196433731-f5cd3659-ea4f-41dc-a138-26224d251c41.png">
<br>

今回は、Technology Zoneの環境にOpenShift環境を払い出して、GitHubに公開されているアプリケーションをデプロイしました。従来の手作業でのデプロイと違って、OpenShiftを利用することでコマンドを数業打ち込むだけで簡単にデプロイをすることができました。
同じTechnology Zoneに用意されているコンテンツでも、付与されている権限の範囲が異なることが分かりました。また1つのマイクロサービスをデプロイするのは少し大変ということだと思いました。ただ、robot-shopが少し特殊なアプリだったので苦戦失敗することもありましたが、通常の簡単なアプリであればSCCの制約は関係なくデプロイをすることができると思います。
