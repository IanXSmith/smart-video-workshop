# Converting a TensorFlow* Model in Linux*

> **Note:** *For Intel® Distribution of OpenVINO™ toolkit release R2 and R3, the file paths used in the TensorFlow* model conversion are different. Please follow the steps carefully for your installed Intel® Distribution of OpenVINO™ toolkit version. You can check the Intel® Distribution of OpenVINO™ toolkit version by going to /opt/intel/ folder. If the folder starts with computer_vision_sdk_2018.2.xxx, it is release R2. If it starts with computer_vision_sdk_2018.3.xxx, it's release R3. Please confirm the version before you follow the steps.*

##### For Intel® Distribution of OpenVINO™ toolkit release R2, use virtual environment for Python* by running:

    source /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/venv/bin/activate
    
You should see “(venv) <user>: “on the command line if you are in the Python* virtual environment
	
##### For Intel® Distribution of OpenVINO™ toolkit release R3, virtual environment for Python* is not required. 
  
### Pre-Requisites: 
#### Change ownership of the directory to the current user (for both R2 and R3)

> **Note:** *replace the usernames below with your user account name*
		
	sudo chown username.username -R /opt/intel
    
#### Install Pre-Requisites for TensorFlow* Framework (for both R2 and R3)

> :warning: Already done for the workshop laptops.

    cd /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/install_prerequisites/
    sudo ./install_prerequistes_tf.sh
   
#### Download Model(s) from TensorFlow* -slim library (for both R2 and R3)
There are a number of pre-trained public models in the TensorFlow*-slim repository. The models are distributed as Python scripts and checkpoint files.
First of all download repository with models.
Note: This is done in home directory(~)

    cd
    git clone https://github.com/tensorflow/models/
    cd models/research/slim

#### Export Inference Graph and download checkpoint file (for both R2 and R3)
Export inference graph for one of the available models using the following command (Inception V1 in this example): 

    python3 export_inference_graph.py --alsologtostderr --model_name=inception_v1 --output_file=/tmp/inception_v1_inf_graph.pb
    
The script creates inference graph file with name “inception_v1_inf_graph.pb” in the /tmp direcory.

#### Download archive with checkpoint file (Inception V1 in this example): (for both R2 and R3)

    export CHECKPOINT_DIR=/tmp/checkpoints
    
    mkdir $CHECKPOINT_DIR
    
    wget http://download.tensorflow.org/models/inception_v1_2016_08_28.tar.gz
    
    tar -xvf inception_v1_2016_08_28.tar.gz
    
    mv inception_v1.ckpt $CHECKPOINT_DIR
    
    rm inception_v1_2016_08_28.tar.gz

#### Freeze the model
The last step is to freeze the graph. To do this you need to know the output node of the model you are planning to freeze. This information is found by running the summarize_graph.

##### Summarize Graph (for both R2 and R3)
Go to ~/models/research/slim/ directory and run summarize_graph.py script.

    cd ~/models/research/slim/
    
    python3 /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/mo/utils/summarize_graph.py --input_model=/tmp/inception_v1_inf_graph.pb

The output layer/node name should be in the last line of text on the console and should look like:

1 input(s) detected:
Name: input, type: float32, shape: (-1,224,224,3)
1 output(s) detected:
InceptionV1/Logits/Predictions/Reshape_1Freeze Graph

##### Freeze the graph for Intel® Distribution of OpenVINO™ toolkit release R2
The script generates inception_v1_frozen.pb file with the frozen model in the directory you are currently in.

	python3 /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/venv/lib/python3.5/site-packages/tensorflow/python/tools/freeze_graph.py --input_graph /tmp/inception_v1_inf_graph.pb --input_binary --input_checkpoint /tmp/checkpoints/inception_v1.ckpt --output_node_names InceptionV1/Logits/Predictions/Reshape_1 --output_graph inception_v1_frozen.pb
 
##### Freeze the graph for Intel® Distribution of OpenVINO™ toolkit release R3
The script generates inception_v1_frozen.pb file with the frozen model in the directory you are currently in.
 
 	python3 /usr/local/lib/python3.5/dist-packages/tensorflow/python/tools/freeze_graph.py --input_graph /tmp/inception_v1_inf_graph.pb --input_binary --input_checkpoint /tmp/checkpoints/inception_v1.ckpt --output_node_names InceptionV1/Logits/Predictions/Reshape_1 --output_graph inception_v1_frozen.pb
    
For Intel® Distribution of OpenVINO™ toolkit release R3, if you might get warning message "Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA" while executing above command, ignore that. 

### Convert Frozen Tensorflow* model to IR using Model Optimizer (for both R2 and R3)
Assuming you are in the ~/models/research/slim/ directory 

    python3 /opt/intel/computer_vision_sdk/deployment_tools/model_optimizer/mo_tf.py --input_model inception_v1_frozen.pb --input_shape [1,224,224,3] --mean_values [1024,1024,1024] --scale_values [128,128,128]

This should produce “inception_v1_frozen.xml” and “inception_v1_frozen.bin” file. The xml file contains the topology information of the model and the bin file contains the model’s weights and biases. These two files are expected when using the inference engine so make note of the path.


### Run Classification Sample (for both R2 and R3)

The classification sample will showcase the Intel® Distribution of OpenVINO™ toolkit inference engine using TensorFlow model Inception_v1_frozen IR files (.xml & .bin) and an input image of a car to classify.
The classification collateral is defined as the input image car_1.bmp, the Inception_v1_frozen IR files (.xml & .bin), and the labels file inception_v1_frozen.labels.
Create a new directory that will hold the classification sample app and all needed components to run the classification sample
Note: The following steps should be followed and are assuming you are following the preceding steps. You should be in the home directory.

    source /opt/intel/computer_vision_sdk/bin/setupvars.sh
 
Navigate to the classification_sample executable file:

    cd /opt/intel/computer_vision_sdk/deployment_tools/inference_engine/samples/build/intel64/Release

Place Classification App collateral in current local directory:

    sudo cp ~/models/research/slim/inception_v1_frozen.* .
    
    sudo cp /opt/intel/computer_vision_sdk/deployment_tools/demo/car_1.bmp  .
    
    sudo cp /opt/intel/computer_vision_sdk/deployment_tools/demo/squeezenet1.1.labels ./inception_v1_frozen.labels

#### Run Application
Note: To see all the flags that the sample takes as input run  ./classification_sample -h

    ./classification_sample -i car_1.bmp -m inception_v1_frozen.xml

Expected Output:


    [ INFO ] InferenceEngine: 
      API version ............ 1.0
      Build .................. 10478
    [ INFO ] Parsing input parameters
    [ INFO ] Loading plugin

      API version ............ 1.0
      Build .................. lnx_20180314
      Description ....... MKLDNNPlugin
    [ INFO ] Loading network files:
      /home/monique/inception_v1_frozen.xml
      /home/monique/inception_v1_frozen.bin
    [ INFO ] Preparing input blobs
    [ WARNING ] Image is resized from (749, 637) to (224, 224)
    [ INFO ] Batch size is 1
    [ INFO ] Preparing output blobs
    [ INFO ] Loading model to the plugin
    [ INFO ] Starting inference (1 iterations)
    [ INFO ] Average running time of one iteration: 12.1089 ms
    [ INFO ] Processing output blobs

    Top 10 results:

    Image /opt/intel/computer_vision_sdk/deployment_tools/demo/car_1.bmp

    829 0.9999994 label streetcar, tram, tramcar, trolley, trolley car
    905 0.0000005 label window shade
    950 0.0000001 label orange
    430 0.0000000 label basketball
    56 0.0000000 label king snake, kingsnake
    370 0.0000000 label guenon, guenon monkey
    490 0.0000000 label chain mail, ring mail, mail, chain armor, chain armour, ring armor, ring armour
    575 0.0000000 label golfcart, golf cart
    41 0.0000000 label whiptail, whiptail lizard
    856 0.0000000 label thresher, thrasher, threshing machine



