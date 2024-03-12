## 概要
　本ドキュメントは、AWSとAzure間のVPN Connectionを構築するためのマニュアルです。<br/>
以下の通り、VPN ConnectionでAWS VGW～Azure VGW間を接続します。<br/>

![vpn-connection-between-aws-and-azure](https://github.com/yamamototis1105/vpn-connection-between-aws-and-azure/assets/114621183/1cb8ebf8-cd67-48b9-b962-900d846821c2)

## 利用方法
### AWSインスタンス作成
1. VPC作成
   * 名前「sample-vpc」、IPv4 CIDRブロックは「10.0.0.0/24」で作成。
1. サブネット作成 
   * 名前「sample-subnet」、アベイラビリティ―ゾーンは「ap-northeast-1a」、IPv4 CIDRブロックは「10.0.0.0/24」で作成。 
1. 仮想プライベートゲートウェイ作成 
   * 名前「sample-vgw」で作成。 
   * VPCへアタッチ。 
1. カスタマゲートウェイ×2作成 
   * 名前「sample-cgw」、ルーティング「動的」、BGP ASN「65001」、IPアドレス「(Azure側のアドレス)」で作成。 
1. サイト間VPN接続作成 
   * 名前「sanmple-vpn」、ターゲットゲートウェイタイプ「仮想プライベートゲートウェイ」で作成。 <br/>
※事前共有鍵、ピアアドレスは予め設計しとくべき。自動の場合、設定をダウンロードのうえ確認する必要あり。 
   * Tunnel1、Tunnel2のピアアドレスを確認。 
1. ルートテーブル更新 
   * 宛先「10.1.0.0/24」、ネクストホップ「(仮想プライベートゲートウェイのID)」を追加。 
1. セキュリティグループ作成 
   * セキュリティグループ名「sample-sg」、説明「sample-sg」で作成。 
   * プロトコル「すべてのICMP」、送信元「0.0.0.0/0」を追加。 
1. EC2インスタンス作成 
   * AMI「Amazon Linux」、インスタンスタイプ「t2 medium」、 ネットワーク「sample-vpc」、サブネット「sample-subnet」、タグ「Name: sample-ec2」、セキュリティグループ「(セキュリティグループのID)」で作成。 
1. SSH用の秘密鍵・共通鍵を作成 
   * ssh-keygen -t rsa 
   * sudo chmod 600 .ssh/id_rsa 
1. ping実行 
   * nohup ping 10.0.0.4 -c 86400 > result & 。
<br/>

### Azureインスタンス作成
1. Vnet作成 
   * 名前「sample-vnet」、地域「(Asia Pacific) Japan East」で作成。 
1. サブネット作成
   * 名前「sample-subnet」、CIDR「10.0.1.0/24」で作成。
1. Gatewayサブネット作成
   * 名前「sample-subnet」、CIDR「10.0.1.0/24」で作成。
1. 仮想ネットワークゲートウェイ作成 
   * 名前「sample-vgw」、地域「Japan East」、ゲートウェイの種類「VPN」、VPNの種類「ルートベース」、SKU「VpnGW1」、世代「Generation1」、仮想ネットワーク「sample-vnet」、パブリックIPアドレス「新規作成」、パブリックIPアドレス名「sample-pip」、アクティブ/アクティブ モードの有効化「無効」、BGPの構成「有効」、自律システム番号「65515」、カスタムの Azure APIPA BGP IP アドレス「169.254.21.2, 169.254.22.2」で作成。 
1. ローカルネットワークゲートウェイ×2作成 
   * 名前「sample-lgw」、地域「Japan East」、エンドポイント「IPアドレス」、IPアドレス「(AWS側のアドレス)」、アドレス空間「10.0.0.0/24」、BGP設定の構成「はい」、自律システム番号「64512」、BGPピアのIPアドレス「169.254.21.1」で作成。 
1. 接続作成 
   * 接続の種類「サイト対サイト(IPsec)」、名前「sample-con」、地域「Japan East」、仮想ネットワークゲートウェイ「sample-vgw」、ローカルネットワークゲートウェイ「sample-lgw」、IKEプロトコル「IKEv2」、AzureプライベートIPアドレスを使用する「なし」、BGPを有効にする「✓」、プライマリBGPアドレスを有効にする「✓」、プライマリカスタムBGPアドレス「169.254.21.2」で作成
1. VMインスタンス作成 
   * 名前「sample-vm」、地域「(Asia Pacific) Japan East」、可用性オプション「インフラストラクチャ冗長は必要ありません」、セキュリティの種類「Standard」、イメージ「Red Hat Enterprise Linux」、VMアーキテクチャ「x64」、ユーザー名「(任意)」、パスワード「(任意)」、OSディスクの種類「Standard HDD(ローカル冗長ストレージ)」、仮想ネットワーク「sample-vnet」、サブネット「sample-subnet」、パブリックIP「なし」、NIC ネットワーク セキュリティグループ「なし」で作成。 

## 関連技術
<img src="https://img.shields.io/badge/AWS-Site_to_Site_VPN-orange"></img> <img src="https://img.shields.io/badge/GCP-HA_VPN-blue"></img>
