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
## 以下の電話番号のパターンを検出します。  
2桁市外局番 0x-xxxx-xxxx  
3桁市外局番 0xx-xxx-xxxx  
4桁市外局番 0xxx-xx-xxxx  
5桁市外局番 0xxxx-x-xxxx  
IP電話 050-xxxx-xxxx  
携帯 070-xxxx-xxxx, 080-xxxx-xxxx, 090-xxxx-xxxx  
## 対象外のパータン  
市外局番の() (0x)-xxxx-xxxx  
フリーダイヤル 0120-xxx-xxx  
国番号含む表現 +81-x-xxxx-xxxx  

## Office 365 での注意事項
### 1. 機密情報のテストでTXT ファイルを利用した場合
Office 365の管理画面での機密情報のテストで、TXT ファイルを利用した場合、半角・全角は区別されて扱われます。
### 2. 全角英数字が半角英数字に事前に変換される
一方で機密情報のテストで Word ファイル等を利用した場合や、Office 365 内のメールやチャット本文内のテキストのマッチングにおいては、全角英数字は半角英数字に変換された上で正規表現とのマッチングが行わます。  
### 3. 全角に挟まれた半角ハイフンの扱い
"全角-全角"のように全角に挟まれた半角ハイフンは前後にスペースが挿入され" - "に置換された上で、マッチングが行われる点に注意します。  

## 電話番号の正規表現
### 採用している電話番号の正規表現
    (?<![a-zA-Z0-9ａ-ｚＡ-Ｚ０-９－ー―\-]| - )[0０]([0-9０-９]([－ー―\-]| - )[0-9０-９]{3}|[0-9０-９]{2}([－ー―\-]| - )[0-9０-９]{2}|[0-9０-９]{3}([－ー―\-]| - )[0-9０-９]|[0-9０-９]{4}([－ー―\-]| - )|[5789５７８９][0０]([－ー―\-]| - )[0-9０-９]{3})[0-9０-９]([－ー―\-]| - )[0-9０-９]{4}(?![a-zA-Z0-9ａ-ｚＡ-Ｚ０-９－ー―\-]| - )
### 上記の解説  
#### [0-9０-９]
全角もしくは半角の数字1桁を示しています。  
#### [0-9０-９]{4}
全角もしくは半角の数字4桁を示しています。  
#### ([－ー―\\-]| - )
全角もしくは半角のハイフンとハイフンに類似する全角文字、注意事項3に留意した変換後の" - "のいずれかを示しています。"\\-"は"\\"によりエスケープしていますが、単に半角のハイフンを示しています。  
#### [5789５７８９]
全角もしくは半角の5,7,8,9のいずれか一文字を示しています。  
#### (?<![a-zA-Z0-9ａ-ｚＡ-Ｚ０-９－ー―\\-]| - )
他の部分がマッチした後、前に半角もしくは全角の英数字やハイフンがないことを確認する正規表現のアサーションとなっており、電話番号の形式を部分的に含む、E03-0000-0000のようなシリアル コードなどの類似する文字列を排除するようにしています。  
#### (?![a-zA-Z0-9ａ-ｚＡ-Ｚ０-９－ー―\\-]| - )
他の部分がマッチした後、後ろに半角もしくは全角の英数字やハイフンがないことを確認する正規表現のアサーションとなっており、電話番号の形式を部分的に含む、03-0000-0000-0000のようなシリアル コードなどの類似する文字列を排除するようにしています。  

# ハイフンの種類
| 文字 | UTF-8 | Unicode Code Point | Description | 今回の対象かどうか |
|:---:|---:|---:|:---|:---:
| - | 2D | U+002D | Hyphen-Minus | x |
| ー | E383BC | U+30FC | Katakana-Hiragana Prolonged Sound Mark | x |
| ‐ | E28090 | U+2010 | Another Hyphen ||
| ‑ | E28091 | U+2011 | Non-Breaking Hyphen ||
| ‒ | E28092 | U+2012 | Figure Dash ||
| – | E28093 | U+2013 | En Dash ||
| — | E28094 | U+2014 | Em Dash ||
| ― | E28095 | U+2015 | Horizontal Bar | x |
| − | E28892 | U+2212 | Minus Sign ||
| ⁃ | E28183 | U+2043 | Hyphen Bullet ||
| ﹣ | EFB9A3 | U+FE63 | Small Hyphen-Minus ||
| ｰ | EFBDB0 | U+FF70 | Halfwidth Katakana-Hiragana Prolonged Sound Mark ||
| - | EFBC8D | U+FF0D | Fullwidth Hyphen-Minus | x |



# 参考情報
1. Web で正規表現のチェックができる[正規表現チェッカー](http://okumocchi.jp/php/re.php)  
1. 文字のコードを確認できる [Unidode文字ツール](https://www.marbacka.net/msearch/tool.php#chr2enc)  
