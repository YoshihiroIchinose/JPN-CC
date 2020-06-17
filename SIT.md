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
以下のパターンを全角数字も考慮して検出します。また数字の桁数が合ってないものを排除するための処理を正規表現で行っています。市外局番なしの電話番号、()での市外局番の表現、フリーダイヤル、+81などの国番号を含むものなどは対象外としています。なおOffice 365の管理画面での機密情報のテストでは、半角・全角は区別されて扱われるものの、メールやチャット本文内のテキストのマッチングにおいては、全角英数字は半角英数字に変換され、"全角-全角"のように全角に挟まれた半角ハイフンは前後にスペースが挿入され" - "に置換された上で、マッチングが行われる点に注意する。

## 電話番号の正規表現
### 採用している電話番号の正規表現
    (?<!([a-zA-Z0-9０-９－ー―\-]| - ))[0０]([0-9０-９]([－ー―\-]| - )[0-9０-９]{3}|[0-9０-９]{2}([－ー―\-]| - )[0-9０-９]{2}|[0-9０-９]{3}([－ー―\-]| - )[0-9０-９]|[0-9０-９]{4}([－ー―\-]| - )|[5789５７８９][0０]([－ー―\-]| - )[0-9０-９]{3})[0-9０-９]([－ー―\-]| - )[0-9０-９]{4}(?!([a-zA-Z0-9０-９－ー―\-]| - ))
