# 日本向け個人情報の定義
Office 365 DLP 向けに日本での個人情報に該当するパターンを定義するものです。

# カスタムの機密情報として定義 XML で取り込み
## PowerShell より Exchange Online に接続
    $UserCredential = Get-Credential
    $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.compliance.protection.outlook.com/powershell-liveid/ -Credential $UserCredential -Authentication Basic -AllowRedirection
    Import-PSSession $Session -DisableNameChecking

## ローカルの XML ファイルをアップロード
    New-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\JPN_SIT.xml" -Encoding Byte)

## 一度アップロード済みで新しいバージョンの定義に更新の場合
    Set-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\JPN_SIT.xml" -Encoding Byte)
    
# DLPのポリシーを設定
取り込んだ以下のカスタムの個人情報を含むファイル等を検出するよう DLP のポリシーを設定のこと。
SIT1.住所  
SIT2.電話番号  

# 住所
## 正規表現
    (?:北海道|東京都|(?:大阪|京都)府|(?:神奈川|和歌山|鹿児島)県|[^\x01-\x7E　]{2}県)[\s　]?[^\x01-\x7E　]{1,6}[市郡区町村][\s　]?[^\x01-\x7E　]
# 電話番号
電話番号の定義では以下のパターンを全角数字も考慮して検出します。また数字の桁数が合ってないものを排除するための処理を正規表現で行っています。
## 固定電話
### 0x-xxxx-xxxx のパターンの正規表現
    (^|[^0-9])0\d-\d{4}-\d{4}($|[^0-9])
### 0xx-xxx-xxxx のパターンの正規表現
    (^|[^0-9])0\d{2}-\d{3}-\d{4}($|[^0-9])
### 0xxx-xx-xxxx のパターンの正規表現
    (^|[^0-9])0\d{3}-\d{2}-\d{4}($|[^0-9])
### 0xxxx-x-xxxx のパターンの正規表現
    (^|[^0-9])0\d{4}-\d-\d{4}($|[^0-9])
## 携帯・IP電話
### 050-xxxx-xxxx, 070-xxxx-xxxx, 080-xxxx-xxxx, 090-xxxx-xxxx のパターンの正規表現
    (^|[^0-9])(050|070|080|090)-\d{4}-\d{4}($|[^0-9])
