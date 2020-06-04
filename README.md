# JPN-CC
こちらのプロジェクトでは、Office 365のCommunication Complianceを利用・検証するに当たって、職場にふさわしくないコミュニケーションを検出するための用語集を提供します。

# カスタムの機密情報として要注意ワードを XML で取り込み
PowerShell より Exchange Online に接続

$UserCredential = Get-Credential
$Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.compliance.protection.outlook.com/powershell-liveid/ -Credential $UserCredential -Authentication Basic -AllowRedirection
Import-PSSession $Session -DisableNameChecking

ローカルに XML ファイルをアップロード

New-DlpSensitiveInformationTypeRulePackage -FileData (Get-Content -Path "C:\Users\imiki\Desktop\Work\Comp\CC_Reference.xml" -Encoding Byte)

# Communication Complianceのポリシーを設定
以下の取り込んだカスタムの機密情報を使ってCommunication Complianceのポリシーを設定のこと

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
