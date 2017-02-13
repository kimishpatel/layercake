# layercake
Tensorflow based deep learning library for CNN and fully connected networks. Allows easy usage and quick exploration of various topologies.

## Usage:

### Import netork and layercake module

Network module is used to read the json file describing the network topology along with few other parameters

```
from layercake import *
from network import *
```

### Initialize instance of a Network class.
```
network = Network(file_name) #file_name is a name of json file describing network e.g. network.json
```
### Set input size and output size:

If input is a 2D/3D image as input to first conv layer then
```
network.set_input_size((image_width, image_height, num_channels))
```
Otherwise:
```
network.set_input_size(input_size)
```
Set output size: This usually describes the number of classes
```
network.set_output_size(num_labels)
```

### Generate tensorflow graph
Once the network topology is read, create an instance of layercake module with the network we just read to generate tensorflow graph
```
cake = LayerCake(network)
```
After this step, cake instance has a graph object that holds tensorflow graph just created. Furthermore it sets up appropriate optimizer, specified in .json file, with specified learning rate.

### Run training
Training can be run on layercake object just created as follows:
```
test_accuracy = cake.run_training(num_iter, dataset.train_dataset, dataset.train_labels, 
                dataset.valid_dataset, dataset.valid_labels, dataset.test_dataset, dataset.test_labels, batch_size, True)
```
num_iter = number of iterations to run training for
train_data, train_labels, valid_dataset, valid_labels, test_dataset, test_labels = training, validation and test dataset and corresponding labels. These are supposed to be numpy arrays. Make sure you data in standardized and the shape of each sample matches the one we specified above.
Call above returns test_accuracy.

### Some caveats

Some parameters as of now are hardcoded. For example, kernel size and stride are hardcoded. I hope to work on these in future. In my experiments on AWS P2 instances I was running out of memory durinv validation and test accuracy evaluation. So I divided them up in batches of 1000 samples to get around the memory issue. You can set this to appropriate size by setting cake.valid_train_step (defaults to 1000).
Right now layercake supports only softmax as output layer and cross entropy for loss. It should not be hard to support other outputs and loss functions but I have not gotten around to it. So this is another piece that is kind of hard coded even though you can specify that in the .json file.

### Example json file
```
{
    "input_size": 0,
    "output_size": 0,
    "slice0": {
        "type" : "conv",
        "layers" : [[3, 3, 1, 64], [3, 3, 64, 96], [5, 5, 96, 128], [5, 5, 128, 256]],
        "activations" : ["relu", "relu", "relu", "relu"],
        "pooling" : ["none", "max", "none", "max"]
    },
    "slice1": {
        "type" : "fc",
        "layers" : [128, 64],
        "activations" : ["relu", "relu"]
    },
    "output": {
        "final" : "softmax",
        "loss" : "cross_entropy"
    },
    "learning_rate" : 0.008,
    "regularization" : 0.005,
    "droput": 0.90,
    "optimizer" : "Momentum",
    "num_iterations": 25001,
    "model_name" : "entire_number"
}
```

### Example usage
In SVHN_train_and_classify.py file one can see the usage of layercake and network module. DO NOT try to run it directly as it requires lot of other things that are not part of this project. It is provided for illustration purpose only.
