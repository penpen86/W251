# HW06
Homework 06 - W251 MIDS Summer 2019

## BERT Toxicity Dataset

On this homework we worked with BERT as a word embedding, and using two types of architecture: *V100* and *P100*. The framework used for running the Neural Networks was *Pytorch* and we used the Adam Optimizer.

### Steps
1. Create the VM instances in IBM cloud
```
//  P100
ibmcloud sl vs create --datacenter=lon06 --hostname=p100a --domain=vinitoxic.com --image=2263543 --billing=hourly  --key=1433621 --flavor AC1_8X60X100 --san
// V100
ibmcloud sl vs create --datacenter=lon04 --hostname=v100a --domain=vinitoxic.com --image=2263543 --billing=hourly  --key=1433621 --flavor AC2_8X60X100 --san
```

2.- From there we ran both notebooks in both machines. In terms of results we had that the V100 is 3 times as fast as the P100, having spend 6 hours and 12 minutes, while the V100 lasted 1 hour and 57 ,imutes

3.- Contents of the repo:

  - Jupyter Notebook from the P100. There, I decided to submit to Kaggle. The submission image is on the repo too
  - Jupyter Notebook from the V100. There, I decided to run the train in 2 epochs, and submit those results to Kaggle (image on the repo)
  - We improved the scoring from one epoch, and jumped 252 spots with my second submission. 

