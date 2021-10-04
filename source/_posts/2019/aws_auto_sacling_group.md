---
title: "[實作筆記] AWS Auto Scaling Group"
date: 2019/08/25 11:21:46
tags:
    - 實作筆記
    - AWS
---

## 前情提要

最近將一個服務掛上雲端(AWS)，使用 EC2 ，
並且實作了 Auto Scaling ，特別紀錄一下。

## 概觀

以下使用的都是 AWS 的服務

1. Auto Scaling Group (ASG) :  
    本篇主角，根據指定的 Policies 與 LC 加開減少機器。

2. Launch Configuration (LC) :  
    一個 ASG 背後一定要搭配一個 LC，可以說是 ASG 的生命共同體，  
    用來指定 EC2 的配置，並透過 `User Data` 指定實體起動時所需要執行的工作。  
    註: 也有使用 Launch Template 的作法，這篇不會討論。

3. Target Group (TG):
    由 1 到多個 EC2 實體(instance)組成，用來與 ELB(ALB) 搭配。

4. ALB :  
    流量負載平衡，與 TG 作為搭配。可以依不同條件轉導到不同 TG

![Auto Scaling Group](/images/2019/8/aws_asg_overview.jpg)

### 組合零件

這有點像是在玩模型玩具，各個部份的小零件可以組成一個大零件，  
再將各個部件結合，完成我所要的功能。
以下是這所需要的各個零件

- Golden Image : 由實際的 EC2 Instance 建立。並包含 Cloud Watch Agent 的配置。
- User Data : Lanuch Configuration 的一個子項目，可以在 EC2 啟動時，執行指定的命令，比如說在 WINDOWS 透過 Powershell 抓取新版程式來建置服務。
- Launch Configuration : 由上述兩個零件 (Gloden Image & User Data) 組成，另外還可以設定 EBS、Plocies 等設定…
- Target Group : 用來存放相關實體(EC2 Instance)的抽象概念, 可以設定 Health Check 來確定實體的健康情況。
- Auto Scaling Group : 透過 Lanuch Configuration 來建立實體，並且可以將實體放入指定的 Target Group 之中。並且可以設定 Policy 來實現 Scaling Out/In.
- ALB : 負載平衡的一種，透過設定，可以將 Request 導到對應的 Target Group。

## 紀錄

### Add CloudWatch Agent

1. Login EC2 Instance
2. 安裝 CloudWatch Agent
3. 準備一個 Json 檔如下

    ```json
    {  
    "agent":{  
        "logfile":"c:\\ProgramData\\Amazon\\AmazonCloudWatchAgent\\Logs\\amazon-cloudwatch-agent.log"
    },
    "logs":{  
        "logs_collected":{  
            "files":{  
                "collect_list":[  
                {  
                    "file_path":"D:\\logs\\**.log",
                    "log_group_name":"/MY/Service/Worker",
                    "log_stream_name":"{instance_id}",
                    "timezone":"UTC",
                    "timestamp_format":"%Y-%m-%dT%H:%M:%S"
                }
                ]
            }
        },
        "log_stream_name":"default-log-stream",
        "force_flush_interval":5
    }
    }
    ```

4. 開啟powershell，切換目錄至 AmazonCloudWatchAgent

    ```bash
    cd "C:\Program Files\Amazon\AmazonCloudWatchAgent"
    ```

5. 執行以下語法

    ```bash
    ./amazon-cloudwatch-agent-ctl.ps1 -a fetch-config -m ec2 -c file:yourjsonfile.json -s
    ```

### 站台加入 Health Check

- 加入靜態檔案(ex:check.html)
- IIS Building *80
![IIS Building* 80](/images/2019/8/aws_iissetting.jpg)
- 設定 Target Group 的 Health Check (ex:/check.html)
![AWS Target Gropu Health Check](/images/2019/8/aws_tg_healthcheck.jpg)

### 包成 Gloden Image

1. 機器下 ALB

    ![機器下ALB](/images/2019/8/aws_gi_out_alb.jpg)

2. 開啟 Ec2LaunchSettings.exe

    ![開啟 Ec2LaunchSettings.exe](/images/2019/8/aws_ec2_launch_settings.jpg)

3. 透過 Ec2LaunchSettings 關機

    ![透過 Ec2LaunchSettings 關機](/images/2019/8/aws_ec2_launch_settings_turn_off.jpg)

4. Waitting Instance Stoped , Create Images

    ![Waitting Instance Stoped , Create Images](/images/2019/8/aws_gi_create_images.jpg)

### Create Launch Configuration

![Create Launch Configuration](/images/2019/8/aws_create_lc.jpg)

1. Choose AMI
    ![Choose Gloden Image](/images/2019/8/aws_choose_ami.jpg)
2. Choose Instance Type
    ![t2.small](/images/2019/8/aws_choose_instance_type.jpg)
3. Configure details
    ![Add User Data](/images/2019/8/aws_userdata.jpg)
4. Add Storage
    ![Set EBS](/images/2019/8/aws_ebs.jpg)
5. Configure Security Group
6. Review

### Create Auto Scaling group

1. Configure Auto Scaling group details
    ![Configure Auto Scaling group details](/images/2019/8/aws_create_asg.jpg)
2. Configure scaling policies
    > 略
3. Configure Notifications
    > 略
4. Configure Tags
    > 略
5. Review

## 參考

- [使用 EC2 執行個體建立 Auto Scaling 群組 - Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/zh_tw/autoscaling/ec2/userguide/create-asg-from-instance.html)
- [Launch Configurations - Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/en_us/autoscaling/ec2/userguide/LaunchConfiguration.html)
- [使用AWS Application Load Balancer](https://aws.amazon.com/cn/blogs/china/aws-alb-route-distribute/)
- [Network Load Balancers 的目標群組](https://docs.aws.amazon.com/zh_tw/elasticloadbalancing/latest/network/load-balancer-target-groups.html)
- [為 Network Load Balancer 建立目標群組](https://docs.aws.amazon.com/zh_tw/elasticloadbalancing/latest/network/create-target-group.html)
- [CloudWatch Logs](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/logs/QuickStartWindows2016.html#configure_cwl_download)
- [使用 CloudWatch 代理程式從 Amazon EC2 執行個體和現場部署伺服器收集指標和日誌 - Amazon CloudWatch](https://docs.aws.amazon.com/zh_tw/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)

(fin)
