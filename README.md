# IBM Technology Zoneを利用して、Red Hat OpenShift環境にアプリケーションをデプロイ
Technology Zoneを利用して、OpenShift環境を作成し、公開されているサンプル・アプリケーションをデプロイして実際に動かしてみました。実施してみて、難しかったところや詰まったところも含めて紹介していきます。

## 目次
1. [Technology ZoneにOpenShift環境を払出し](https://github.com/topfieldy/OpenShift-Lab/blob/main/README.md#technology-zone%E3%81%ABopenshift%E7%92%B0%E5%A2%83%E3%82%92%E6%89%95%E5%87%BA%E3%81%97)
2. [サンプル・アプリケーションのデプロイ](https://github.com/topfieldy/OpenShift-Lab#github%E3%81%AB%E5%85%AC%E9%96%8B%E3%81%95%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92openshift%E4%B8%8A%E3%81%AB%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4)
3. [デプロイしたアプリケーションにアクセスする](https://github.com/topfieldy/OpenShift-Lab#%E5%AE%9F%E9%9A%9B%E3%81%AB%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E9%96%8B%E3%81%84%E3%81%A6%E3%81%BF%E3%82%8B)
4. [終わりに](https://github.com/topfieldy/OpenShift-Lab#%E7%B5%82%E3%82%8F%E3%82%8A%E3%81%AB)

## 1. Technology ZoneにOpenShift環境を払出し
OpenShiftの環境の予約方法を解説します。  
1. [Technology Zone](https://techzone.ibm.com/)へのアクセス  
Technology Zoneにアクセスし、IBMidを利用してログインします。

2. 利用する環境の検索と決定  
今回は[Red Hat OpenShift on IBM Cloud basics lab - Environment](https://techzone.ibm.com/collection/roks-basics-lab#tab-1)の環境を利用しています。
画面キャプチャ左側の環境です。`Reserve`→`Reserve now`→`Submit`の順にクリックすると注文の詳細設定の画面に遷移します。
<img width="1237" alt="スクリーンショット 2022-10-17 10 18 16" src="https://user-images.githubusercontent.com/112134163/196212028-8fc8eff2-14d8-480d-acb7-bacc85bc1161.png">

3. 環境を作成するためのフォームの入力  
`Purpose`は下記から選択する必要があります。普段自分の学習用に使う場合は、無条件で72時間まで使える`Practice/Self-Education`を選ぶことが多いです。その他、利用したいデータセンターや利用開始日時等を入力します。
　　<img width="1065" alt="スクリーンショット 2022-10-17 10 21 27" src="https://user-images.githubusercontent.com/112134163/196212759-0ad01696-f6cc-40bf-8c93-5de787dd3a41.png">

4. アカウントの招待の受け入れ  
約30分ほどで環境がプロビジョニングされ、メールで完了通知とTechnology ZoneのIBM Cloudアカウントからアカウントへの招待の通知がきました。
招待を受け入れることで、Technology Zoneに作成したOpenShift環境へアクセスできるようになります。  
※アカウントへの招待メールが届かない場合は、プロビジョニング完了メールに記載されている下記のような文章の中の`HERE`からアカウントへの招待を受け入れてください。
<img width="685" alt="スクリーンショット 2022-10-19 23 21 18" src="https://user-images.githubusercontent.com/112134163/196718102-ec179878-c53b-4c99-9486-ece9272d1711.png">


5. プロビジョニングされたOpenShift環境の稼働確認  
プロビジョニング完了メールか、Technology Zoneの`My Reservation`から作成された環境を確認することができます。
<img width="1073" alt="スクリーンショット 2022-10-18 13 51 52" src="https://user-images.githubusercontent.com/112134163/196338566-d733b5bc-b53a-47bc-b3ac-9aa45620d39a.png">

## 2. サンプル・アプリケーションのデプロイ  
実際にGitHubに公開されているサンプル・アプリケーションをデプロイしていきます。  
監視用アプリケーションの[Instanaのアプリ Robot-shop](https://github.com/instana/robot-shop)を使用します。
<img width="1440" alt="スクリーンショット 2022-10-18 0 14 14" src="https://user-images.githubusercontent.com/112134163/196215705-763f36ef-b356-4983-b1ae-3bb6bb53e464.png">  

### 前提条件1 OpenShift CLI(ocコマンド)のインストール
[IBM CloudのDocsのガイド](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift-cli&locale=ja)を参考にして、作業しているPC上にocコマンドをインストールしました。


### 前提条件2 Helmのインストール
Helmとは、リポジトリからのインストールや、Helmによってデプロイされたアプリの管理をCLIで簡易化するためのOSSです。Kubernetes向けのパッケージマネージャーとしてよく使われるようです。LinuxにおけるyumやRPM、MacOSにおけるHomebrewなどと同様のツールと考えると理解しやすいです。Helmを作業しているPC上にインストールしておきます。


### デプロイの手順
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
この`export KUBECONFIG`というコマンドで環境変数を設定することでクラスターへのアクセス情報を設定しているが、Red Hat OpenShift on IBM Cloudのサービスの場合は、この方法だけでは、ログインできないので、サービスが規定しているocコマンドでログインしています。

このコマンドでは、OpenShiftクラスターへログインをしています。私の場合は以下の方法でログインをしました。  
1. `OpenShiftクラスター`の`OpenShift Webコンソール`にアクセス
2. 画面右上にある自分のアカウント名の右側にあるプルダウンから`ログインコマンドのコピー`
3. `Display Token`をクリック
4. `Log in with this token` のコマンドをコピー（※前提条件1が必須）
5. CLIにコピーしたログインコマンドをペーストすることでログインできる  


```
oc adm new-project robot-shop
```
`oc`とはOpenShiftのコマンドで、`adm`はAdministratorの権限を使って、`robot-shop`という新しいProjectを作成しています。  
※`robot-shop`はProjectの名前なので、自分が識別できる名前であれば、他の名前で問題ありません。
<br>
<br>

```
oc adm policy add-scc-to-user anyuid -z default -n robot-shop
oc adm policy add-scc-to-user privileged -z default -n robot-shop
```
OpenShiftではデフォルト状態で「restricted」というSCC(Security Context Constraints)のセキュリティ制約がかかっています。SCCとはPodのパーミッションを制御する機能です。
今回のアプリは制約が強いので、デフォルトのユーザーに特別な強い権限を付与しています。  
ただ、今回使用したOpenShift環境では付与されている権限が弱く、下記のようにErrorとなってしまい、Projectに強い権限を付与することができませんでした。 結果として、権限が足りなくて、この先のHelmでインストールするところでアプリのデプロイに失敗しました。この操作を行うためには「cluster-admin」ロールに対応したIBM Cloud IAM役割の付与が必要です。

```
oc adm　 policy add-scc-to-user anyuid -z default -n robot-shop
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io "system:openshift:scc:anyuid" is forbidden: User "IAM#yuki.uehara2@ibm.com" cannot get resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "robot-shop"

oc adm policy add-scc-to-user privileged -z default -n robot-shop
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io "system:openshift:scc:privileged" is forbidden: User "IAM#yuki.uehara2@ibm.com" cannot get resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "robot-shop"  
```
<br>

ここからは強い権限が設定されている別の[Technology ZoneのOpenShift環境](https://techzone.ibm.com/collection/custom-roks-vmware-requests)に切り替えて進めていきます。  
手順としては、ここまで実施したことと同様に進めるだけなので割愛します。 
<br>
<br>

```
cd robot-shop/K8s
helm install robot-shop --set openshift=true -n robot-shop helm
```
cdコマンドで、Helmで導入するパッケージのリソースが含まれる`K8s`ディレクトリに移動し、次の行で、パッケージをインストールしています。これでデプロイ完了です。  
※前提条件2 に記載したとおり、事前にHelmクライアントをインストールしておかないとコマンドが実行できないので注意が必要です。 
<br>
<br>


### デプロイされたアプリを確認
`OpenShiftのDeveloperコンソールのトポロジー`からデプロイされているリソースを確認することができます。  
Robot-shopを構成するマイクロサービスがデプロイされているのが確認でき、アプリケーションのデプロイは成功しているようです。  
<img width="1148" alt="スクリーンショット 2022-10-18 13 44 26" src="https://user-images.githubusercontent.com/112134163/196337451-52a1bbb0-0a5f-4cd5-9203-da998ca04af7.png"> 
<br>
<br>

## 3. デプロイしたアプリケーションにアクセスする
アプリのアクセス方法は、`トポロジー`のデプロイしたアプリの中から、`web`を選び、`リソース`のタブを開きます。`webのPodポートが「8080」`だと確認できます。次にサービスの中の`web`のリンク先にアクセスします。
<img width="1078" alt="スクリーンショット 2022-10-18 21 38 55" src="https://user-images.githubusercontent.com/112134163/196431554-322e68da-9e9f-49d7-b037-9828934da5ea.png"> 

`外部ロードーバランサーのIPが「163.73.69.51」`と確認できます。
<img width="1070" alt="スクリーンショット 2022-10-18 21 42 38" src="https://user-images.githubusercontent.com/112134163/196432390-85425722-871d-4e0f-bd9b-fba3ab399ba4.png">   
<br>

ブラウザに下記のようにIPアドレスを入力します。
```
http://<外部ロードバランサーのIP>:<web Podポート>
http://163.73.69.51:8080
```
その結果しっかりと動いていることが確認できました。  
下記が、アクセスした画面です。
![スクリーンショット 2022-10-20 16 14 54](https://user-images.githubusercontent.com/112134163/196881272-c19b0031-75cc-4edd-85f5-013e865a444b.png)
<br>
`Login/Register`にアクセスでき、フォームへの入力もできています。
![スクリーンショット 2022-10-20 16 17 04](https://user-images.githubusercontent.com/112134163/196881703-1b96108f-3d51-4dfe-b8e6-b12a60ddf50f.png)
<br>
`Categories`という項目にもアクセスできることが確認できました。
![スクリーンショット 2022-10-20 16 18 23](https://user-images.githubusercontent.com/112134163/196881961-739456c3-1c26-49d2-8999-4f6014629d59.png)
<br>

## 4. 終わりに
今回は、Technology Zoneの環境にOpenShift環境を払い出して、GitHubに公開されているサンプル・アプリケーションをデプロイしました。従来の手作業でのデプロイと違って、OpenShiftを利用することでコマンドを数業打ち込むだけで簡単にデプロイをすることができました。  
また、同じTechnology Zoneに用意されているコンテンツでも、付与されている権限の範囲が異なるというのも気付きとしてありました。1個1個のマイクロサービスのデプロイとは違い、マイクロサービス全部を束ねた1つのアプリとしてデプロイするのは少し大変でした。ただ、robot-shopが少し特殊なアプリだったので苦戦失敗することもありましたが、通常の簡単なアプリであればSCCの制約は関係なくデプロイをすることができると思います。
