
# Homework 09

In this homework, we are using distributed learning for training a Machine Translation algorithm between German and English, and we'll use [BLEU](https://en.wikipedia.org/wiki/BLEU) as our source of performance. 

## Troubleshooting

### No Distributed Learning

The first step was to spin to V100 VM with 2 GPU each on IBM Cloud and connect them together using `mpirun`. The main problem was that no matter how many tries, IBM cloud only provisioned a 1 GPU V100 on **server B**. Thus, the first time that I tried to run `nohup mpirun --allow-run-as-root -n 4 -H <vm1 private ip address>:2,<vm2 private ip address>:2 -bind-to none -map-by slot --mca btl_tcp_if_include eth0 -x NCCL_SOCKET_IFNAME=eth0 -x NCCL_DEBUG=INFO -x LD_LIBRARY_PATH python run.py --config_file=/data/transformer-base.py --use_horovod=True --mode=train_eval &` an error occurred on the `nohup` file and the python script ran only on one VM (**server A**). Interestingly enough, this training was faster than the distributed one (~30 hours for 300k steps) with a similar **BLEU Graph**

![BLEU - 1 VM](https://github.com/penpen86/W251/blob/master/HW09/Images/bleu_onegpu.JPG)

This training was later downloaded to the Jetson as *no-dist* training (I think it will be a great exercise to run the Lab with both checkpoints and compare the performance). This training is the first 95,000 lines of the **nohup.out**)

### Distributed Learning

After spotting the network error, I proceed to try to connect them together via `mpirun`. Because ibmcloud was uncooperative, I decided to distribute the learning using 3 GPUs instead of 4. Using this command `nohup mpirun --allow-run-as-root -n 3 -H 10.45.103.102:2,10.45.103.104:1 -bind-to none -map-by slot --mca btl_tcp_if_include eth0 -x NCCL_SOCKET_IFNAME=eth0 -x NCCL_DEBUG=INFO -x LD_LIBRARY_PATH python3 run.py --config_file=/data/transformer-base.py --use_horovod=True --mode=train_eval &`, where I made a server of 3 nodes, 2 from V100a and 1 from V100b. Using this configuration I was able to run the script in a distributed way, created layers on the 3 GPUs. This ran for almost 16 hours, but it was too slow: the network was the bottleneck and I was able to only run 6000 steps.

![Network Sample - No Upgrade](https://github.com/penpen86/W251/blob/master/HW09/Images/network_sample.JPG)

I proceeded to upgrade the network on ibmcloud I was able to decrease the time between each step from ~33 seconds to ~1.5 seconds

![Network Sample - Upgrade](https://github.com/penpen86/W251/blob/master/HW09/Images/network_sample01.JPG)

This configuration ran for around ~60 hours, until ~160k steps, where it was stopped manually. It was also downloaded to the Jetson as *dist* training. This part of the training is the last part of the **nohup.out** file, starting from line 98,128

#### Graphs from Distributed Learning

* BLEU Score

  ![BLEU](https://github.com/penpen86/W251/blob/master/HW09/Images/bleu_distributed.JPG)
 
* Global gradient

  ![Global Gradient](https://github.com/penpen86/W251/blob/master/HW09/Images/global_gradient.JPG)
  
* Eval Loss

  ![Eval Loss](https://github.com/penpen86/W251/blob/master/HW09/Images/eval_loss.JPG)
  
* Learning Rate

  ![Learning Rate](https://github.com/penpen86/W251/blob/master/HW09/Images/learning_rate.JPG)
  
* Train Loss

  ![Train Loss](https://github.com/penpen86/W251/blob/master/HW09/Images/train_loss.JPG)


#### Distributed Learning Questions

1. Time: 
    * One GPU was particularly fast: 300k steps around ~30 hours. 
    * Distributed learning without a network upgrade: ~33 sec per step, thus 300k will take around ~2750 hours
    * Distributed learning with a network upgrade: ~1.5 sec per step, thus 300k will take around ~125 hours
    
2. Fully trained? Looking at the BLEU graph, it's still moving. The training loss is tending to a minimum, so after the process is finished, probably it will be fully trained, or at least close to it. 

3. Overfitting? The Eval loss tends to an almost constant value after just 80k, which means we are somewhat overfitting the evaluation set.

4. GPU Utilization
  * For one GPU training, the GPU utilizations was hovering 82%, which I found strange.
  * For fistributed traing, the the 3 GPU where fully used
  
  **V100a**
  
  ![GPUa](https://github.com/penpen86/W251/blob/master/HW09/Images/gpu_usage.JPG)
  
  **V100b**
  
  ![GPUb](https://github.com/penpen86/W251/blob/master/HW09/Images/gpu_usagev100b.JPG)
  
5. Network? Without improvement, the network is a bottleneck, with improvement, it's not (as shown before on the Troubleshooting section)

6. Learning Rate? The model is using the learning rate as a hyperparameter to train, and it goes from full attention to inattention. That means that the memory of the LSTM is changing while we train, changing the level of context we are looking at each step. For later steps, we use smaller learning rate to have better backpropagation.

7. Training size? 

  **Lines**
  
  ![Lines](https://github.com/penpen86/W251/blob/master/HW09/Images/lines_data.JPG)
  
  **Memory (MB)**
  
  ![Size](https://github.com/penpen86/W251/blob/master/HW09/Images/size_data.JPG)
  
8. Checkpoint: It contains the path of the model, with all previous paths. Each path contains the metadata, the index of the weights, the losses and the best models.

9. Size of the checkpoint:
  * 1 GPU: 4610 MB (this training was completed)
  * Distributed: 4339 MB (this trining was stopped after 160k steps)
  
10. Step? It was answered before, but as a summary:
  * 1 GPU: 0.33 sec
  * Distributed without network upgrade: 33 sec
  * Distributed with network upgrade: 1.5 sec
  
11. The correlation between network speed and step duration is negative. The higher the network, the smaller the duration in seconds of the steps, as we can see from the previous point. 
