# 日本向け個人情報の定義
Office 365 DLP 向けに日本での個人情報に該当するパターンを定義するものです。現時点で住所と電話番号を定義しています。用語は、カスタムの機密情報の定義として [XML](https://github.com/YoshihiroIchinose/JPN-CC/blob/master/JPN_SIT.xml) ファイルで用意していますので、ダウンロードの上、PowerShell で Office 365 に取り込んで利用ください。他にも日本向けの Communication Compliance 用の用語集の定義はこちらの[ページ](https://github.com/YoshihiroIchinose/JPN-CC/blob/master/README.md) で紹介しています。

# カスタムの機密情報として XML を取り込み
## PowerShell より Exchange Online に接続
    $UserCredential = Get-Credential
    $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.compliance.protection.outlook.com/powershell-liveid/ -Credential $UserCredential -Authentication Basic -AllowRedirection
    Import-PSSession $Session -DisableNameChecking

## ローカルの XML ファイルをアップロード
    New-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\JPN_SIT.xml" -Encoding Byte)

## 一度アップロード済みで新しいバージョンの定義に更新の場合
    Set-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\JPN_SIT.xml" -Encoding Byte)
    
# DLPのポリシーを設定
取り込んだ以下のカスタムの個人情報定義を利用して、Office 365のDLP のポリシー等を設定下さい。それぞれ該当するものが見つかれば信頼度 80 でマッチします。  
SIT1.住所  
SIT2.電話番号  

# 住所
以下の正規表現で都道府県＋市区町村＋全角文字で構成される文字列をマッチングし日本での住所の表現を検出します。
## 正規表現
    (?:北海道|東京都|(?:大阪|京都)府|(?:神奈川|和歌山|鹿児島)県|[^\x01-\x7E　]{2}県)[\s　]?[^\x01-\x7E　]{1,6}[市郡区町村][\s　]?[^\x01-\x7E　]
# 電話番号
以下のパターンを全角数字も考慮して検出します。また数字の桁数が合ってないものを排除するための処理を正規表現で行っています。市外局番なしの電話番号、()での市外局番の表現、フリーダイヤル、+81などの国番号を含むものなどは対象外としています。なお設定の都合上、共通するハイフン+数字4桁の部分を見つけて、さらにパターンごとの詳細の一致を確認するようにしています。

## 電話番号の正規表現
### 半角での電話番号の正規表現
    0(\d-\d{3}|\d{2}-\d{2}|\d{3}-\d|\d{4}-|(5|7|8|9)0-\d{3})\d-\d{4}
### 全角での電話番号の正規表現
    ０([０-９][-－ー―][０-９]{3}|[０-９]{2}[-－ー―][０-９]{2}|[０-９]{3}[-－ー―][０-９]|[０-９]{4}[-－ー―]|(５|７|８|９)０[-－ー―][０-９]{3})[０-９][-－ー―][０-９]{4}
### 上記をマージして前後にハイフンや数字が続いていない指定を追加した今回利用している正規表現
    (?<![0-9０-９-－ー―])(0(\d-\d{3}|\d{2}-\d{2}|\d{3}-\d|\d{4}-|(5|7|8|9)0-\d{3})\d-\d{4})|(０([０-９][-－ー―][０-９]{3}|[０-９]{2}[-－ー―][０-９]{2}|[０-９]{3}[-－ー―][０-９]|[０-９]{4}[-－ー―]|(５|７|８|９)０[-－ー―][０-９]{3})[０-９][-－ー―][０-９]{4})(?![0-9０-９-－ー―])
## 参考　検出対象とした固定電話のパターン
### 0x-xxxx-xxxx のパターンの正規表現
    0\d-\d{4}-\d{4}
    ０[０-９][-－ー―][０-９]{4}[-－ー―][０-９]{4}
### 0xx-xxx-xxxx のパターンの正規表現
    0\d{2}-\d{3}-\d{4}
    ０[０-９]{2}[-－ー―][０-９]{3}[-－ー―][０-９]{4}
### 0xxx-xx-xxxx のパターンの正規表現
    0\d{3}-\d{2}-\d{4}($|[^0-9])
    ０[０-９]{3}[-－ー―][０-９]{2}[-－ー―][０-９]{4}
### 0xxxx-x-xxxx のパターンの正規表現
    0\d{4}-\d-\d{4}
    ０[０-９]{4}[-－ー―][０-９][-－ー―][０-９]{4}
## 参考 検出対象とした携帯・IP電話のパータン
### 050-xxxx-xxxx, 070-xxxx-xxxx, 080-xxxx-xxxx, 090-xxxx-xxxx のパターンの正規表現
    0(5|7|8|9)0-\d{4}-\d{4}
    ０(５|７|８|９)０[-－ー―][０-９]{4}[-－ー―][０-９]{4}
