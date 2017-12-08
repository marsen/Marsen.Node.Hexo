---
title: "AWS EC2開機筆記"
date: 2017/12/08 16:42:42
tag:
  - AWS
  - EC2
---
## 應該知道的事
- 這個是教育訓練的筆記
- 使用web console 建立ec2
- 使用cli 建立ec2
- 2017的筆記可能會隨時間變得沒有參考價值
- 關鍵參數都打馬賽克,沒有牽扯到EBS/S3/VPC
- 對你可能沒有幫助

## Web Console

1. login AWS

2. 進入EC2
![](https://i.imgur.com/hRFwjzr.jpg)

3. Launch Instance
![](https://i.imgur.com/g9vlacA.jpg)

4. 選擇AMI(Amazon Machine Image )
![](https://i.imgur.com/dVKPsAp.jpg)

5. 選擇Instance Type(有錢隨便選,沒錢選t2.nano)
![](https://i.imgur.com/61gG2pd.jpg)

6. 設定 Instance Details
![](https://i.imgur.com/NkbKrzL.jpg)

7. 如果想在開機的時候自動安裝一些程式,可以在`Advanced Details` 加語法
windows AMI請用 `Powershell`
![](https://i.imgur.com/bJxWlgd.jpg)

8. 加硬碟
![](https://i.imgur.com/MP9igLc.jpg)

9. 加tag
![](https://i.imgur.com/xDTx2nv.jpg)

10. 設定Configure Security Group
![](https://i.imgur.com/wximWw1.jpg)

11. 預覽與啟動
![](https://i.imgur.com/6Y4fcOI.jpg)

12. 最後一步,選擇key-pair 
![](https://i.imgur.com/fRhUafI.jpg)




## CLI command

```
aws ec2 run-instances --image-id ami-da9e2cbc --count 1 --instance-type t2.nano --subnet-id subnet-cxxxxxxx --user-data file://userdata.sh --tag-specifications "ResourceType=instance,Tags=[{Key=Environment,Value=AWS-Training},{Key=Name,Value=AWS-Training_MarkLin}]" --security-group-ids sg-XXXXXX --key-name marktest.japan.training
```

## 參考文章
- [run-instances](http://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html)

(fin)
