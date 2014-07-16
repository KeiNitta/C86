
***********************************************
Immutable Infrastructureの最適解を探る(chapter用)
***********************************************


Immutable Infrastructureの最適解を探る
=====================================

本特集では、2014年のインフラ界のバズワードである、「Immutable Infrastructure」(以下、IIと略します)について取り上げます。
IIが出てきた経緯や、実際にIIを実践するための方法などについて触れていきたいと思います。


まず、こちらをご覧ください
-------------------------------

.. figure:: img/appprotweet.eps
  :scale: 80%
  :alt: appprotweet
  :align: center

  やめて！！

心臓に何かが刺さった音がしましたか？

次にこちらをご覧ください
----------------------

.. figure:: img/condel.eps
  :scale: 50%
  :alt: condel
  :align: center

  いつまで手動でデプロイしているんですか？

  ＿人人人人人人人人人人人人人人人人人人人人人人＿
  ＞　いつまで手動でデプロイしているんですか？　＜
  ￣Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^Y￣

マサカリが投げられましたね。この本については、後ほど触れることにしましょう。


あわせて読みたい
---------------

CHad Fowler氏 [#iichad]_ の「サーバを捨てて、コードを焼き付けろ！」[#iitys]_ [#iitys2]_ というブログ記事があります。
その記事のJunichi Niino氏による邦訳 [#iihottan]_ を引用してみましょう。

  開発者として、あるいはしばしばシステム管理をする者として、これまで経験したもっとも恐ろしいものの1つは、長年にわたり稼働し続けてなんどもシステムやアプリケーションのアップグレードを繰り返してきたサーバだ。

  なぜか。その理由は、古いシステムはつぎはぎのような処置がされているに違いないからだ。障害が起きたときに一時しのぎのハックで対処され、コンフィグファイルをちょこっと直してやり過ごしてしまう。「あとでChefの方に反映しておくよ」とそのときは言うけれど、炎上したシステムの対処に疲れて一眠りしたあとでは、そんなことは忘れてしまうだろう。

  予想もしないところでcronのジョブが走り始めて、よく分からないけれどなにか大事な処理をしていて、そのことを知っているのは関係者のうちの1人だけとか。通常のソースコード管理システムを使わずにコードが直接書き換えられているとか。システムがどんどん扱いにくくなっていき、手作業でしかデプロイできなくなるとか。初期化スクリプトがもはや、思いも付かないような例外的な操作をしなければ動かなくなっているとか。

  もちろんOSは（きちんと管理されているならば）なんども適切にパッチが当て続けられているだろうが、そこには（訳注：やがて秩序が失われていくという）エントロピーの法則が忍び込んでくるものだし、適切に管理されていないとすればいちどもパッチは当てられず、もしこれからパッチを当てようものならなにが起きるか分からない。

  私たちはこの問題を解決するために何年ものあいだ、チームポリシーの策定から自動化までさまざまな方法を試してきた。そしていま試している新しい方法が「Immutable Deployments」（イミュータブル・デプロイメント）だ。

.. [#iichad] https://twitter.com/chadfowler
.. [#iitys] 邦題は@naoya氏の「Immutable Infrastructure Conference #1」の発言から引用
.. [#iitys2] 「Trash Your Servers and Burn Your Code: Immutable Infrastructure and Disposable Components」http://chadfowler.com/blog/2013/06/23/immutable-deployments/
.. [#iihottan] このへんの流れは、 Junichi Niino氏の「『Immutable Infrastructure（イミュータブルインフラストラクチャ）と捨ててしまえるコンポーネント』 チャド・ファウラー氏」http://www.publickey1.jp/blog/14/immutable_infrastructure.html　を参考にしました。っていうかほぼそのまま

どんどん変更を加えていったことによって、その全容を知る者がいなくなってしまったシステムの誕生です。
そんなシステムはデプロイ職人によってアップデートされることがあり、素人が触るとだいたい失敗します。


継続的デリバリー
---------------

先ほど、「いつまで手動でデプロイしているんですか？」というマサカリを投げてきた本は「継続的デリバリー 信頼できるソフトウェアリリースのためのビルド・テスト・デプロイメントの自動化」 [#iikz]_ (以下、「継続的デリバリー」と略します)です。この本は、ソフトウエアをユーザにできるだけ早く届ける方法が書かれています。つまり書いたコードのテストを自動で行うための手法から、本番環境への安全で素早いデプロイ方法などについて書かれています。

* 手動で変更を加えていったサーバのプログラムのアップデートを行うために、なぜ毎週、戦々恐々としなくてはならないのか？
* バグを出してしまったが、来週のアップデートまで待たせるのか？

本来は、バグを潰したコードを、すぐにでも本番サーバに安全にデプロイしたい、と思っているんじゃないでしょうか。そして、こう考えます。

手動で変更を加えていったサーバは壊そう！

.. [#iikz] http://www.amazon.co.jp/dp/4048707876


作って壊す、そして自動化
----------------------

Martin Fowler氏のブログに、PhenixServer [#iifs]_ という記事があります。不死鳥のように蘇るサーバという意味です。
お仕事で動作中のサーバの監査行ったとき、本番と同じサーバを作ろうとしたところ、構成のズレやアドホックな変更でサーバの設定が「drift」(漂流)していたそうです [#iisfs]_ 。
だったらいっそのこと定期的にサーバを焼き払ったほうがよく、puppetやchefを使ってサーバを作り直そうと書かれています。

.. [#iifs] http://martinfowler.com/bliki/PhoenixServer.html
.. [#iisfs] そんなサーバのことを SnowflakeServer(雪の欠片サーバ) という http://martinfowler.com/bliki/SnowflakeServer.html

あるいは、開発環境をいじくりまくって、やっぱりもとの綺麗さっぱりした状態にもどしたい、なんて経験は一回や二回、いやもっとあったかな？
そんなときに、もし作りなおすことが簡単にできたらどうでしょう。

ここで、先ほどでてきた「継続的デリバリー」の中でも重要な事として **自動化** が何度も登場します。
自動化を推し進めると、コードのテストから、バグの修正や機能の拡張を本番サーバにデプロイするまでがほぼ自動となり、デプロイの回数を安全に増やすことができます。

2012年に行われたカンファレンス、AWS re:Inventにて「Amazonは1時間に最大1000回もデプロイする」 [#iideploy]_ という講演がありました。
そのなかで、「Amazon.comでは11秒ごとに新しいコードがデプロイされている。そして最も多いときで1時間に1079回デプロイが行われた。
これには機能追加だけでなくバグフィクスなども含まれるが。平均で1万、最大で3万ものホストがデプロイを受け取る」とあります。
これは、バグはすぐに潰され、機能の追加の恩恵も受けられることを示します。このサイクルを行うために、継続的デリバリーでも強調されている **自動化** が必須となります。

例えば、この本の原稿の生成も自動化されています [#iikonohon]_ 。
githubにReST形式の原稿をpushすると、それを検知したjenkinsがsphinx [#iisphinx]_ のコマンドを実行し、入稿用のPDFが生成されます。

自動化の最先端として、githubにpull requestを行うとテストが実行され、そのあと本番環境へデプロイされる仕組みが@naoya氏のブログで紹介されています [#iighedep]_ 。
pull requiestをIRCなどのツールで自動化して作成し、Pull Request内容を確認、mergeするとそのままテストが走り、そして本番環境へコードが入ります。
自動化できるところは自動化しましょう。人的ミスがなくなります。

.. [#iideploy] http://www.publickey1.jp/blog/12/amazon11000_aws_reinventday2_am.html
.. [#iisphinx] ドキュメントビルダーのsphinxです。http://sphinx-users.jp/
.. [#iighedep] GitHub 時代のデプロイ戦略 http://d.hatena.ne.jp/naoya/20140502/1399027655
.. [#iikonohon] ななかInsidePRESS vol.1では原稿はGitHubにあり、PDFは手動でビルドしていました 
.. [#iivps] Virtual Private Server。仮想専用サーバのことです。この原稿PDFはさくらのVPSでビルドされています


そうはいっても
^^^^^^^^^^^^^^

確かに壊して作りなおすと言っても、いまさらできないよ・・・時間があればできるけど、それをやっている隙がないということもあるでしょう。
そいういう場合は、人間が毎回ルーチンで行っていることを自動化しましょう。たとえばコードのテストの自動化であったり、デプロイの準備などです。
いつか来る、すべてのシステムの作り直しの時がくるまでに準備しておきましょう [#souhaittemo]_ 。

.. [#souhaittemo] 作り直しの時がこないって？そんなシステムは老朽化がきて、サービスをやめようという判断になるので、そのまま捨てましょう（ぇー


サーバのセットアップの一般的手順
-----------------------------

IIの説明をするまえに、我々は何を自動化したいのかを明確にしておきましょう。例えばサーバのセットアップの一般的手順を示すと下記のようになります [#iisetup]_ 。

* データセンターにサーバを設置してケーブリング。またはインスタンスを立ち上げ
* OSをインストール [#iigoldenimage]_ 
* ミドルウエアをインストールして設定ファイルを書く
* プログラムをデプロイ
* プログラムの動作を確認
* 監視ツールに登録
* DNSに登録
* LBに登録

.. [#iisetup] Serf という Orchestration ツール #immutableinfra http://www.slideshare.net/sonots/serf-iiconf-20140325 の14ページを参考にしました
.. [#iigoldenimage] ゴールデンイメージってやつもあるけど各自ぐぐってね！


Immutable Infrastructure を導入
-------------------------------

いよいよ本題のIIに入ります。

IIの三層
--------

とっつきやすいのでIIの三層の話から入ります。mizzyさんの記事 [#iimi1]_ で三層の話がでてきます。この記事の参照先 [#ii3lay1]_ のPDF [#ii3lay2]_ を引用します [#ii3lay3]_ 。

.. [#iimi1] インフラ系技術の流れ - Gosuke Miyashita - http://mizzy.org/blog/2013/10/29/1/
.. [#ii3lay1] Provisioning Toolchain: Web Performance and Operations - Velocity Online Conference - March 17, 2010 - O'Reilly Media - http://en.oreilly.com/velocity-mar2010/public/schedule/detail/14180
.. [#ii3lay2] Open Source Provisioning Toolchain - http://cdn.oreillystatic.com/en/assets/1/event/48/Provisioning%20Toolchain%20Presentation.pdf
.. [#ii3lay3] このスライドは、もともとToolchainの話をしています。Toolchainとはソフトウエアを作る生産ラインみたいなものです。たとえば「emacs->autoconf->autoheader->automake->libtool->gcc->ld」

.. figure:: img/3layer.eps
  :scale: 50%
  :alt: 3layer
  :align: center

  IIの三層

サーバをセットアップする生産ラインとしてこの３つの層がでてきます。矢印の方向に向かって、ベルトコンベアのようにサーバがセットアップされる様子を表しています。

* Bootstrapping

  * OSのインストールやVM,クラウドのイメージの起動
  * 使われるツールやソフトウエア：Kickstart, Cobbler, OpenStack, AWS

* Configuration

  * ミドルウエアのインストールや設定
  * 使われるツールやソフトウエア：Puppet, Chef, AWS OpsWorks, Ansible

* Orchestration
  
  * アプリケーションのデプロイ
  * 使われるツールやソフトウエア：Fabric, Capistrano, MCollective

どの層で何をやるかは、正確な定義はないので好きなようにしましょう。使われるツールからやれることを想像してみてください。ただし、どの層で何をやるのか決めておかないと手間が増えます。たとえば、kickstartでOSのユーザを作って、さらにChefでも同じユーザを作ろうとしてレシピがコケるとか [#iisurf00]_ 。

.. [#iisurf00] Orchestrationからしれっと Surf を消してますが、まあ無視しましょう

以上は三層で終わっていますが、本誌ではそれに付け加えて２つの層を設定します。

* Test

  * デプロイされたプログラムの動作を確認
  * 使われるツールやソフトウエア：Serverspec

* Agent
  
  * 外部サービスに自分を登録
  * 使われるツールやソフトウエア：Serf

どうでしょうか [#ii]_ 。ここまでくると、先ほどの「サーバのセットアップの一般的手順」を網羅できましたね！ [#iitaechan]_ [#iiyarukoto]_

.. [#ii] このTestとAgentをOrchestrationに含めてもいいんですけどOrchestrationが頭でっかちになるんですよね [脳内調べ]
.. [#iitaechan] やったねタエちゃん、やること増えるよ！！
.. [#iiyarukoto] 初期コストかけて自動化の状態に持って行ってそこからあとは楽になる...と考えていた時期がありました(このへん、かなり大きな問題だったり...)


早速実践してみよう
----------------

そういえばサーバのセットアップの一般的手順で「データセンターにサーバを設置してケーブリング」を自動化してませんよね？えっ？GoogleかAmazonあたりが革新なソリューションを発表してくれることを期待してここでは放置しましょう [#iicable]_ 。

.. [#iicable] このへんのソリューションを作ったら売れそうな感じしますよね。ってかなんで21世紀になってサーバとスイッチを有線でつなぐの？ありえないんですけどーーぷんすか（落ち着いて下さい
.. [#iirack1] っていうかさーなんで21世紀になって電源タップからサーバに電源ケーブル繋がないといけないの？ケーブルが絡みついてあられもない格好に（なりません
.. [#iirack2] そもそもなんでサーバ設置しないといけないの？てかもう、サーバラックとサーバを一体型にしてデータセンターにポンを置けばもう使えるとかできないの？？
.. [#iirack3] ↑このシステム、売れそうな気がするんですけど誰かやってくれないですかねえ。あ、できたら筆者に分け前ください!!シクヨロ!!

さて、IIの三層+二層をひと通り実践してみましょう。Bootstrappingから始まると思った?残念!!Serverspecちゃんでした!! [#iizansaya]_ 

.. [#iizansaya] 残念さやかちゃん。まえがきでこのネタを使おうと準備してたけど結局使えなかったのでここで満を持して登場!!

なんでServerspecから始めるのかだって？それはそこそこ重要で取っ付きやすいからです。サーバのデプロイはchefでもAnsibleでもbashスクリプトでも手動でコマンドを打てば構築はできます。
問題はそのあとです。誰がどうやって、そのサーバが正しくセットアップできているか調べるのか？それにはServerspecを使いましょう。

.. tip::

   この本を作っている第七開発セクションが前回頒布した「ななかInside PRESS vol.4」でChefを特集しました。そのChefを執筆した人曰く、Chefのレシピを書くのが辛くなってきたそうです。
   社内でいろいろなプロジェクト(プロダクト)があります。それらに対応する汎用的なレシピを書くと、設定することが多くなり、扱いづらくなるという現象が起きました。

   そのため、すでにあるレシピをプロダクト担当のインフラの中の人が各自forkして使いやすいように手を加えました。構築に一回使うだけだしいいよね、ってことで一回だけ実行される死屍累々のレシピが作られていったそうです。おしまい。
   
   なお、この話はフィクションです。フィクションですよ！！大事なことなので二回言いました。


動作確認するためにServerspec
^^^^^^^^^^^^^^^^^^^^^^^^^^

Serverspec [#iiscurl]_ とは、ruby製のツールで、Rspec [#iirspec]_ を拡張したものです。ssh経由でOSの内部の状態をチェックすることができます。さっそく具体例を見ていきましょう。
Serverspecのチュートリアルをクリアするといくつかファイルが出来ます。そのとき、テストを記述するspecファイルもサンプルとして一緒に作成されます。

.. code-block::ruby

   require 'spec_helper'
   
   describe package('httpd') do
     it { should be_installed }
   end
   
   describe service('httpd') do
     it { should be_enabled   }
     it { should be_running   }
   end
   
   describe port(80) do
     it { should be_listening }
   end
   
   describe file('/etc/httpd/conf/httpd.conf') do
     it { should be_file }
     its(:content) { should match /ServerName www.example.jp/ }
   end

やってることはフィーリングでなんとかして下さい。え？なんとかならない？しょうがないにゃあ。このspecファイルは、httpdに関連したファイルで、パッケージがインストールされているか、httpdがOS起動時に起動しているか、プロセスが上がっているか、80番ポートをlistenしているかなどをチェックします。なお、localhostにsshで入れる設定であれば、自分自身もテストできます [#iijibun]_ 。

チュートリアルで作ったこのテスト(specファイル)は、1つのサーバに対応しています。複数のサーバをまとめてチェックするものがないかなーと探していたらありました [#iiscd]_ [#iiscdbun]_ 。使ってみましょう。

.. code-block:: sh

   $ git clone git@github.com:dwango/serverspecd.git
   $ cd serverspecd
   $ bundle

hosts.ymlにホスト名とチェックするrolesを書いて、attributes.ymlにroleに与えるパラメーターを書きます。
たとえば自分が所有しているvpsにテストをかけてみましょう。まずは、sshでノンパスで入るために``.ssh/config``を設定。公開鍵は別途登録して下さい。

.. code-block:: conf

   Host nico
     HostName        niko.example.com
     Port            2525
     User            nico_yazawa
     IdentityFile    ~/.ssh/id_rsa
     User            nico

attributes.yml.templateとhosts.yml.templateをリネームしてhosts.ymlを変更。こんな感じ。

.. code-block:: sh

   $ cp attributes.yml.template attributes.yml
   $ cp hosts.yml.template hosts.yml
   $ cat hosts.yml
   nico:
     :roles:
        - os
   maki:
     :roles:
        - os
        - network

設定を見てみましょう。サーバの一覧が並びます。

.. code-block:: sh

   % rake -T                              
   (in /home/chiba/repo/serverspecd)
   rake serverspec       # Run serverspec to all hosts
   rake serverspec:maki  # Run serverspec to maki
   rake serverspec:niko  # Run serverspec to niko

テスト実行してみます。成功したテストは ``.``  、失敗したテストは ``F`` で表示されます。失敗したテストの理由が表示されます。どんなコマンドを実行したか出るので、デバックするときに使います。

.. code-block:: sh

   $ rake serverspec:niko
   (in /home/chiba/repo/serverspecd)
   /usr/local/bin/ruby -S rspec spec/os/os_spec.rb
   .FFFFFFFFF..FF...F.F....FFFFFF........F.........FF..FF..FFFF....F....F..F.......FF....F...FFFFF......FFF
   
   Failures:
   (以下略)

なお、attributes.ymlのosのセクションにパラメータが、テストは ``spec/os/os_spec.rb`` にあります。phpやmysqlのテストも同梱したので、使いたい人は使ってやって下さい。

Serverspecで重要なのは、何をテストするかということです。なるべく重複するテストの数を少なくするのがおすすめです。これをチェックすれば、複数の項目がチェックできるテストが良いです。
応用としては、開発サーバや本番サーバのSAN値 [#iisanti]_ のチェックをしてみましょう。
具体的には、Jenkins [#iijenkins]_ おじさんを使って1日1回程度テストを回して、入ってはいけないパッケージを見つけたり、別のサーバへの疎通ができているかをチェックしましょう [#iiscn]_ 。
テストを書くのはだるいですが、一度やっておけば、バグや障害を検出することができますので、是非やりましょう。

.. [#iiscurl] http://serverspec.org/
.. [#iirspec] http://rspec.info/
.. [#iiscd] https://github.com/dwango/serverspecd 「d」とついているからといって、デーモンではありません
.. [#iiscdbun] bundleコマンドがなければ、``gem install bundler`` でインストールして下さい。``gem`` がなかったらrubyをインストールして下さい
.. [#iijibun] 自分自身といっても人ではなく、サーバのことです。自分のテストは健康診断にでも行って下さい(執筆時期が丁度そんな時期)
.. [#iisanti] SAN値とは、正気度を表すパラメーターのことである - http://dic.nicovideo.jp/a/san値
.. [#iijenkins] http://jenkins-ci.org/ Jenkins CI。継続的デリバリーには必須のアイテム。トリガーを設定してテストなどを実行できるソフトウエアです。実行の結果がわかりやすいです
.. [#iiscn] スイッチやロードバランサの設定がいつのまにか変わっていて疎通できない！(・ω・＼)SAN値!(／・ω・)／ピンチ!なんてことがないように


構築にはAnsible
^^^^^^^^^^^^^^^

構築を自動で行ってくれるソフトウエアといえば chef が有名になってきました。 chefについては、弊サークルが前回頒布した「ななかInside PRESS vol.4」で特集をしているのでご覧ください。
同じものを取り上げても面白くないので、ポストchefになりつつある [#iiann]_ Ansible [#iiansible]_ を取り上げます。さきほど取り上げた、IIの三層の「Configuration」の部分のソフトウエアです。
なお、ここではLinux上でのAnsibleを解説します。Ansible 1.7からWindowsもサポートされたようなので、必要であればドキュメント [#iianwin]_ をご覧ください。

.. [#iiann] 脳内調べ
.. [#iiansible] http://www.ansible.com/home
.. [#iianwin] http://docs.ansible.com/intro_windows.html

Ansibleの利点として、「数時間で自動化できてとってもシンプル！」「構築先のサーバはノンパスsshで入れるようにしておけばOK！」「パワフル」[#iianpo]_ 
さて、何ができるかよくわからないまま使ってみましょう。対象のホストへsshでノンパスでログインできるようにしておけば準備完了。

.. [#iianpo] どの辺がパワフルなのか実はよーわからん

* Ansibleのインストール

Amazon EC2では、これでインストール完了。最新版のAnsibleがインストールされます。

.. code-block:: sh

   $ sudo easy_install pip
   $ sudo pip install ansible

なお、``python-simplejson``パッケージが必要なので、CentOSの古いバージョンでやるときには注意してください。EPELが入っているなら、`` sudo yum install ansible``でインストールできます。pipなら、``sudo easy_install simplejson``でいけるはず。
 [#iiansdo]_ 。

.. [#iiansdo] DigitalOcenan の CentOS 7 では、``yum install -y gcc python-devel`` してから ``sudo easy_install pip && sudo pip install ansible && sudo easy_install simplejson``という感じで ``ansible`` コマンドが起動はしました


  * ansibleとは
  * 使ってみる
  * 利点欠点
  * 参考

    * 不思議の国のAnsible – 第1話 : http://demand-side-science.jp/blog/2014/ansible-in-wonderland-01/


仮想化そのいち Vagrant
^^^^^^^^^^^^^^^^^^^^^

* vagrantとは

  * Hashicorpのやつ
  * VirtualBoxのイメージを作成するツール
  * VMwareでも可
  * Boxと呼ばれるイメージを拾ってきてその中に入ってるOSを起動する
  * Boxはつくれる！かわいいは正義

* 使ってみる

  * DigitalOceanつかってみよう

* 参考

  * 仮想環境構築ツール「Vagrant」で開発環境を仮想マシン上に自動作成する : http://knowledge.sakura.ad.jp/tech/1552/
  * Windows7にVirtualBoxとVagrantをインストールしたメモ : http://k-holy.hatenablog.com/entry/2013/08/30/192243 
  * 1円クラウド・ホスティングDigitalOceanを、Vagrantから使ってみる : http://d.hatena.ne.jp/m-hiyama/20140301/1393669079


仮想化そのに docker
^^^^^^^^^^^^^^^^^^

* dockerとは

  * chrootのつよいやつ
  * OS上にコンテナを作って、そのうえに環境をつくる
  * 差分が重要らしい
  * ネットワークまわりとか、ディレクトリ関連がどうなるのかわからん。chrootでよくね？
  * FAQ形式で掘っていくのもよいかもね。じゃがいもよろしくー

* 使ってみる


Cobbler
^^^^^^^^^

* kickstartはわかっている！環境つくるのめんどいんだよねー向けな人


flynn
^^^^^^

Surf
^^^^^^



その他の問題
------------


ログの管理どうする？
^^^^^^^^^^^^^^^^^^^

* fluentdを使って収集しましょう。いつでもサーバを壊せる状態にしておきましょう。
* Elasticsearch + kibanaでログを可視化できてはっぴー☆


DBどうするよ？
^^^^^^^^^^^^^^

* 気軽に壊せないので、こわさない。以上解散！


サーバの監視
^^^^^^^^^^^^^^^^^^^^

* 気軽にこわせて気軽に立ち上がるサーバに名前をつけると大変なことに！！！
* サーバに名前を付けることは悪であるという議論
* hobbitとかzabbixとかそういうツールだと登録してるホストがなくなるとデータがなくなっちゃうんだよねー過去のトレンドが消えてしまうことが問題
* mackerelを取り上げる。


CI as a Service
-----------------

* まだよくわかってない


まとめ
-------

* 本当にやりたいことは何だ？

  * 実際には運用に入ったサーバを作って壊す富豪的な環境ってあんまりないよね　お金もかかるし。オンプレミスだったらそんな余裕はないはず
  * 運用に入ったサーバの変更を安全にやるためにはどうする
  
* 現在進行形でみんな手探り状態
* おじさんのchef疲れ
* やりたいことを実現するためのツールが乱立している
* 新旧ツールをうまく組み合わせて事故のないデプロイをしていこう！

* インフラでの旨味。構築がミスなく簡単にできる。最初に乗り越えるハードルが高い。よく考えていないとハードルだらけになる。導入コスト
* プログラミングしている側からの便利さ。すぐに環境が作れる。テストの自動化。本番でのバグが少なくなる
* 開発環境DevOps
* 本番環境DevOps


注目すべきトレンド
-----------------

* どくだんとへんけん
* hashicorp http://www.hashicorp.com/blog
* kief morris http://kief.com/
* Martin Fowler http://martinfowler.com/
* chad fowler http://chadfowler.com/
* 英語だけど翻訳すればよめなくはない。雰囲気をつかもう


参考文献
--------
「継続的デリバリー 信頼できるソフトウェアリリースのためのビルド・テスト・デプロイメントの自動化」アスキー・メディアワークス,2012
「WEB+DB PRESS vol.81」技術評論社,2014


IIやる人はこれだけは最低限みておけリンク
------------------------------------

* 今さら聞けない Immutable Infrastructure - 昼メシ物語 / http://blog.mirakui.com/entry/2013/11/26/231658

  - IIについての話題をコンパクトにまとめている良記事。ただしIIはここで出てこないトピックもたくさんある



とりまとめついてない
------------------

* 必要なければdevopsに触れなくていっかなー
* 設定が漂流する。そこにIIを導入していくコスト。cultureは？
* IIが出てきた根源的な点はどこか？メリットが上回るものなのか？現状維持ではダメなのか？何故ダメになったのか？

みなおしする点
-------------

* Serverspecの綴りは、Sが大文字ですね
* 冪等性触れる


壮大なメモ
----------

* PhenixServer : http://martinfowler.com/bliki/PhoenixServer.html

  * フェニックスサーバ。認証監査をしようと思った

    * 今動いている本番環境を再度構築しなおすことになる
    * 定期的にサーバを焼き払ったほうがいい
    * サーバは灰の中から不死鳥のように蘇る。だからフェニックスサーバという
    * 構成のズレ、アドホックな変更でサーバの設定が漂流する。SnowflakeServersにいきつく

      * http://kief.com/configuration-drift.html Configration Drift

    * このような漂流に対向するためにpuppetやchefをつかってサーバを同期し直す。
    * netflixはランダムにサーバを落として大丈夫か試している（ひー

* SnowflakeServer : http://martinfowler.com/bliki/SnowflakeServer.html

  * スノーフレークサーバ。雪のかけらサーバという存在
  * OSやアプリケーションにパッチを当てたりする必要がある
  * 設定を調査すると、サーバによって微妙に違う
  * スキー場にとっては良いが、データセンターではよくない
  * スノーフレークサーバは再現が難しい
  * 本番での障害を開発環境で再現させても調査できない
　
    * 参考文献・目に見えるOpsハンドブック　http://www.amazon.com/gp/product/0975568604
   
  * 芸術家はスノーフレークを好むのだそうだ　http://tatiyants.com/devops-is-ruining-my-craft/
　
    * （サーバ含めそのなかのアプリケーションも工業製品なんだよ！！！わかったか！！！（横暴
    * （昔はひとつのサーバでなんとか出来たけど、今はアクセスも増えてサーバも増えたので芸術品はいらない！！
    * （どーどー落ち着けー、なーー
　
  * スノーフレークのディスクイメージを造ればいいじゃんという議論
  * だがこのディスクイメージはミスや不要な設定も一緒に入っている
  * しかもそれを変更することもある。壊れやすさの真の理由となる（雪だけに
  * 理解や修正がしにくくなる。変更したら影響がどこに及ぶかわからない
  * そんなわけで古代のOSの上に重要なソフトウエアが動作している理由である
  * スノーフレークを避けるためにはpuppetやchefを使って動作の確認のとれたサーバを保持すること
  * レシピを使用すつと、簡単に再構築できる。または、イメージデータを作れる
  * 構成はテキストファイルだから変更はバージョン管理される

  * nologinにしてchefなどからレシピを実行すれば、変更はすべてログに残り監査に対して有効
  * 構成の違いによるバグを減らし、全く同じ環境をつくれる。また、環境の違いに起因するバグを減らせる

    * 継続的デリバリーの本に言及する　あっ

* ConfigurationSynchronization : http://martinfowler.com/bliki/ConfigurationSynchronization.html

  * あんまり重要じゃない

* ImmutableServer : http://martinfowler.com/bliki/ImmutableServer.html

  * やっともどってこれた。この文章からスノーフレークとフェニックスサーバに飛んでいる
  * Netflixが実は実戦でやってたみたい　AMIつくってそれをAWS上に展開している

    * http://techblog.netflix.com/2013/03/ami-creation-with-aminator.html
    * AMIを作るツール　https://github.com/Netflix/aminator#readme


* WEB+DB PRESS 81からメモ

  - IIデメリット　サーバが立ち上がった状態からの変更を禁じているのでちょっとした変更を入れるのにもサーバを作りなおす必要がある
  - サーバの生成廃棄コストが頻繁にあると運用コストが増大する
  - サーバの作成や廃棄が簡単なクラウドを使うのが楽
  - ホストの生成廃棄プロセスをAPIでやれると楽。LBとかもAPIでやれると楽
  - クラスタ監視ツールにmackerel.ioを使おう
  - dokku , flynn, apache mesos, Surf
  - pakker
  - BGDepではLBをAPIで変更できると楽