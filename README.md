# Office 365 Communication Compliance 用、要注意サンプル用語集
こちらのプロジェクトでは、Office 365のCommunication Complianceを利用・検証するに当たって、職場にふさわしくないコミュニケーションを検出するためのサンプルの用語集を提供します。用語は、カスタムの機密情報の定義として [XML](https://github.com/YoshihiroIchinose/JPN-CC/blob/master/CC_Reference.xml) ファイルで用意していますので、ダウンロードの上、PowerShell で Office 365 に取り込んで利用ください。なお 2020 年 6 月の時点では、Office 365 の Exchange/Teams/Skype for Busines/Yammer のうち、Exchange でのみ日本語でのキーワードの検出を確認しています。その他ソースの対応は、Office 365 での日本語ワードブレーカーのアップデートを待つ必要があります。他にも日本向けの住所や電話番号など個人情報の定義はこちらの[ページ](https://github.com/YoshihiroIchinose/JPN-CC/blob/master/SIT.md) で紹介しています。

# 事前に一度実施
## PowerShellを管理権限で立ち上げて以下を実行
    Import-Module ExchangeOnlineManagement
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned

# カスタムの機密情報として要注意ワードを XML で取り込み
## PowerShell より Exchange Online に接続
    Connect-IPPSSession

## ローカルの XML ファイルをアップロード
    New-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\CC_Reference.xml" -Encoding Byte)

## 一度アップロード済みで新しいバージョンの定義に更新の場合
    Set-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\CC_Reference.xml" -Encoding Byte)

# Communication Complianceのポリシーを設定
取り込んだ以下のカスタムの機密情報を含むコミュニケーションを検出するよう Communication Compliance のポリシーを設定のこと。なおそれぞれのキーワードのみの検出の場合、精度 60 以上を対象とし、付随ワード含む検出を対象とする場合は精度を 80 以上で設定のこと。

W1.働き方要注意ワード  
S1.顧客満足関連  
S2.クレーム関連  
P1.業務外の連絡  
P2.度重なる食事の誘い  
H1.パワハラ  
H2.性急な要求  
L1.違法  
L2.カルテル  
Q1.検査偽装  
Q2.外為法  

# 定義済みの NG ワード
## W1.働き方要注意ワード
### 内容
サービス残業や、過度な労働に関連した用語。
### キーワード (信頼度60)
"サービス残業","サビ残","無休","無給","無償","早朝出勤","休日出勤","労基","労働基準","奴隷","ただ働き","徹夜","強制","週末","土日","祝日"  
### 付随ワード (キーワードとセットで信頼度80) 
"仕事","業務","残業","ボランティア","勤務"  

## S1.顧客満足関連
### 内容
お客様からの感謝などポジティブな反応に関連した用語。
### キーワード (信頼度60)
"感謝","いいね","感動","ありがとう","良かった","喜ばれて","お礼"  
### 付随ワード (キーワードとセットで信頼度80) 
"お伝え","お客様"  

## S2.クレーム関連
### 内容
お客様からのクレームなどネガティブな反応に関連した用語。
### キーワード (信頼度60)
"怒って","激怒","憤慨","クレーム","苦情"  
### 付随ワード (キーワードとセットで信頼度80) 
"返品","キャンセル","退室","退席","手違い","間違い","不満","不信","品質","落ち度","障害","不具合","再発","予期せぬ","土下座","温度感"

## P1.業務外の連絡
### 内容
業務外と断った上での社内への連絡およびコミュニケーション。
### キーワード (信頼度60)
"業務外","仕事とは関係ない","業務中"  
### 付随ワード (キーワードとセットで信頼度80) 
"広範囲","失礼","恐れ入","恐縮","破棄","無視","個人的","すいません"  

## P2.食事の誘い
### 内容
職場における食事や飲みの誘いなどに関するコミュニケーション。
### キーワード (信頼度60)
"飲み","夕食","ディナー","食事","いい店","晩飯","晩御飯","一杯"  
### 付随ワード1 (キーワードとセットで信頼度80) 
"誘","行かない","どう","行く","行きませんか","いく","いきませんか"  
### 付随ワード2 (キーワードとセットで信頼度90)  
"何度も","しつこ","やめて","止めて","断り”,"いや","嫌","イヤ","二人","無視"  

## H1.パワハラ
### 内容
社内での叱責や侮辱および攻撃的なコミュニケーション。
### キーワード (信頼度60)
"クズ","バカ","馬鹿","アホ”,"あほ","役立たず","無能","頭がおかしい","使えない","死ね","死んでくれ","目障り","給料泥棒"  
### 付随ワード (キーワードとセットで信頼度80) 
"無駄","むだ","ムダ","クビ","くび","失格","左遷","常識","当たり前"

## H2.性急な要求
### 内容
社内での性急な要求に関するコミュニケーション。
### キーワード (信頼度60)
"緊急","直ちに","速やかに","早急に","すぐさま","至急"  
### 付随ワード (キーワードとセットで信頼度80) 
"連絡","対応","処置","回答","応答","返信"  

## L1.違法
### 内容
法律違反や犯罪に言及したコミュニケーション。
### キーワード (信頼度60)
"犯罪","法律違反","違法","違憲","イリーガル","違反","殺人","横領","背任","裏切り"  
### 付随ワード (キーワードとセットで信頼度80) 
"すれすれ","スレスレ","ぎり","ギリ","グレー","アウト","セーフ","行為"  

## L2.カルテル
### 内容
価格や数量の調整に関するコミュニケーション。カルテルとは関係のない内容も多くヒットするため精査が必要。
### キーワード (信頼度60)
"卸値","単価","価格","数量","カルテル","談合"  
### 付随ワード (キーワードとセットで信頼度80) 
"調整","方針","合意","協議","合意","利益","利ざや","入札","応札","同等品","情報交換","面談","販売","卸","取引","協力金","賛同金"  

## Q1.検査偽装
### 内容
データの改竄や偽装などに言及したコミュニケーション。
### キーワード (信頼度60)
"偽装","不正","改竄","改ざん","捏造","リコール"  
### 付随ワード (キーワードとセットで信頼度80) 
"不適合","無資格者","監査","品質","検査結果","故意","意図的","見過ごす","書き換え","指示"  

## Q2.外為法
### 内容
外為法に合致する武器に転用可能な物品に言及したコミュニケーション。
### キーワード (信頼度60)
"武器","兵器","大量破壊","核","超電導","セラミック","工作機械","ロボット","集積回路","半導体","暗号装置","センサー","ソナー","レーダー","レーザー","航法装置","ジャイロスコープ","潜水艇","無人"  
### 付随ワード (キーワードとセットで信頼度80) 
"キャッチオール","ワッセナー","規制","化学","生物","ミサイル","ロケット","原子力","外為","転用","海外","外国","輸出","為替"  
