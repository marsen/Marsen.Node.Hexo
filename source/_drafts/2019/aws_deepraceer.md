---
title: "[實作筆記] AWS DeepRacer"
date: 2019/05/18 15:11:38
tag:

---

## 概念
- [Introduction to Reinforcement Learning](https://d2k9g1efyej86q.cloudfront.net/)

### Reinforcement Learning

強化學習(Reinforcement Learning , 簡稱 RL),  
就好像在訓練寵物坐下或握手一樣，  
透過獎勵(Reward)特定的行為(Action)來達到訓練的目標，  

### DeepRacer 的角色
#### Agent
在比賽中是真實的縮小迷你車(1:18)，車上帶有鏡頭可以拍攝路況， 
在訓練中則是使用模擬器,可以模

- Enviroment
- Action
- Reward
- State


## 實作

1. 建立資源

2. 預設的 獎勵方程

```py
def reward_function(params):
    '''
    Example of rewarding the agent to follow center line
    '''
    
    # Read input parameters
    track_width = params['track_width']
    distance_from_center = params['distance_from_center']
    
    # Calculate 3 markers that are at varying distances away from the center line
    marker_1 = 0.1 * track_width
    marker_2 = 0.25 * track_width
    marker_3 = 0.5 * track_width
    
    # Give higher reward if the car is closer to center line and vice versa
    if distance_from_center <= marker_1:
        reward = 1.0
    elif distance_from_center <= marker_2:
        reward = 0.5
    elif distance_from_center <= marker_3:
        reward = 0.1
    else:
        reward = 1e-3  # likely crashed/ close to off track
    
    return float(reward)
```

## 除錯

錯誤訊息

> Error in IAM role creation
> Please try again after deleting the following roles: AWSDeepRacerServiceRole, 
> AWSDeepRacerSageMakerAccessRole, AWSDeepRacerRoboMakerAccessRole, 
> AWSDeepRacerLambdaAccessRole, AWSDeepRacerCloudFormationAccessRole.

這是 IAM Roles 已經存在, 通常是你已經建立過了這些 Roles , 只要刪除了就可以了

## 使用資源與服務
- DeepRacer
- Reinforcement learning >　Reinforcement learning
- CloudWatch > Log Groups 
- IAM
  - AWSDeepRacerServiceRole 
  - AWSDeepRacerSageMakerAccessRole 
  - AWSDeepRacerRoboMakerAccessRole
  - AWSDeepRacerLambdaAccessRole
  - AWSDeepRacerCloudFormationAccessRole.

### 如何清除
- DeepRacer > 按下 Reset

## 參考
- [AWS 機器學習戰鬥營](https://aws-workshop-tw-03.splashthat.com/)
- [aws-samples/aws-deepracer-workshops](https://github.com/aws-samples/aws-deepracer-workshops/tree/master/Workshops/2019-AWSSummits-AWSDeepRacerService/Lab1)
- [Introduction to Reinforcement Learning](https://d2k9g1efyej86q.cloudfront.net/)
- [Train and Evaluate AWS DeepRacer Models Using the AWS DeepRacer Console - AWS DeepRacer](https://docs.aws.amazon.com/en_us/deepracer/latest/developerguide/deepracer-console-train-evaluate-models.html)

(fin)