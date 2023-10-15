项目名称：智能停车场系统 Intelligent Parking System
### Evidence of User Need
分析用户需求，如表格所示：

| User Need            | Problem Analysis                                                                                                                                                   | AI Advantage                                                 |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| 寻找车位困难         | 目前停车场针对这个问题基本的解决思路就是在车位上用灯或其他方式来表明此处是否有空位，但是这种方式有局限性，比如附近车位都满了的时候就难以判断往哪走能更快地找到车位 | 使用AI能够统筹整个停车场的数据，然后帮助用户规划出完整的路线 |
| 难以判断哪个车位合适 | 在进入比如商业中心的地下停车场时，司机也许具备去某个指定店铺或者区域的需求，但是地下停车场的标识通常很难判断自己所找的位置是否合适                                 | 使用AI模型能够帮助司机根据目的地找到最合适的停车位           |
| 停车场管理困难       | 目前传统停车场管理能够做到的统计基本上就是场内车辆数量统计以及监控，但这并不能完全令人满意                                                                         | AI模型统计能够实时监控到停车场内的每辆车，达到精细管理的目标 |
| 其他需求             | 除上述之外还有一些简单需求比如用户会忘记自己车停在哪了等，这些问题用传统的方式也可以解决                                                                           | 但是使用AI系统能够将大大小小的功能集中到一起，方便用户使用   | 

综合上述需求及分析，我们认为一个智能停车场系统能够有效解决司机和停车场管理方在使用和管理停车场时所遇到的问题。
### Augmentation versus Automation
智能停车场系统主要有两方面的用户，分别是进入停车场的**司机**和停车场的**管理方**。
- 对于司机而言，使用该停车场系统能够帮助其找到更加合理的车位（例如更加靠近电梯厅，更加方便司机去其指定的目的地等），并且帮助司机记住车停在什么位置，车停了多久等等信息，这是augmentation。同时，智能停车场系统能够自动生成前往最有停车位的路径，免去了司机自己逐个寻找的过程，这属于是automation的帮助。不过综合来说，对于司机用户，该智能停车场系统的帮助主要体现在augmentation上。
- 对于管理方而言，智能停车系统能够提供传统的管理方案（通常为进出车辆统计和视频监控）所无法提供的信息，比如指定车牌的车辆目前位于哪里，或者停车场当前停车情况，是否需要疏通等，这些是augmentation。而更加明显的是一个智能停车场系统能够自动完成全部的数据统计以及管理工作，并且可以在需要人工干预的时候发出提示，如此一来整个停车场在常规状态下可以实现自动化的运行，这是AI带来的automation方面的帮助，对于管理方而言，智能停车场系统更多地是带来automation方面的帮助。
### Design Your Reward Function
智能停车场系统除了使用AI技术的功能之外，还有一些传统的功能，例如对于司机帮助记录停车位在什么地方并提供地图引导，以及对于管理方记录车辆的出入统计功能等。这些功能不使用AI技术，也不需要训练等步骤，所以不纳入设计reward function的讨论，但是对于这些功能的准确性要求基本上为完全正确。
我们针对智能停车场系统的两大主要功能：对司机用户的车位引导功能和对管理用户的车辆追踪功能进行reward function设计，分别如下表：

|          |                                   Positive                                   |                                                                      Negative                                                                       |
| -------- |----------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| Positive |                **TP** <br/> 引导到了司机满意且最为合适的车位                 | **FN** <br/> 车位选择不够好，车位可能不是最优的或者不能令司机用户满意 <br/> 路径选择有问题，绕路或进行了错误引导 <br/> 引导到了已经停了车的错误车位 |
| Negative | **FP** <br/> 系统略过了良好的停车位 <br/> 系统对某些合适的停车位却提示不合适 |                                                   **TN** <br/> 判断出某些车位不适合司机选择并合理提醒                                                   | 

关于车位引导功能的结论：Our AI model will be optimized for **precision**, because drivers have the ability to make judgments and determine whether a parking space is suitable, however, it can be quite troublesome if the system guides the driver to the wrong parking spot. We understand that the tradeoff for choosing this method means our model will be more likely to provide available but suboptimal guidance suggestions.

|          | Positive                                      | Negative                                      |
| -------- | --------------------------------------------- | --------------------------------------------- |
| Positive | **TP** <br/> 正确追踪指定车辆（车牌指定）         | **FN** <br/> 指定车辆未被还在停车场内但未被追踪到 |
| Negative | **FP** <br/> 追踪到了错误的车辆或者其他错误的目标 | **TN** <br/> 追踪指定车辆时仅获取正确车辆信息，其他车辆都被识别为负样本                                              |

关于管理方车辆追踪功能的结论：Our AI model will be optimized for **recall**, because the problem of untracked vehicles can be compensated by multiple detections, but if the wrong vehicle is tracked, it can likely cause unnecessary trouble. We understand that the tradeoff for choosing this method means our model will be more prone to missed detections.
### Define Success Criteria
针对我们的智能停车场系统，我们设计了一下的几个success criteria：
1. 如果车位引导功能的precision达不到95%以上，我们还需要对模型进行进一步的训练，即便降低引导建议质量也要确保不推荐或引导错误的车位。
2. 如果管理方的车辆追踪功能recall达不到95%以上，我们就还需要针对停车场固定场景的追踪模型进一步进行训练，需要确保停车场中的车辆都能被追踪到。
除上述AI方面的criteria以外，针对实际软件系统，还有一些别的评价标准：
1. 如果系统稳定性达不到持续运行的标准，那么就需要对通信稳定性、是否有备用服务器等方面进行检查。
2. 系统的延迟如果过高，例如在常规规模的停车场如果需要超过20s才能给出车辆引导建议，那么我们就需要对AI算法计算的速度进行改进。