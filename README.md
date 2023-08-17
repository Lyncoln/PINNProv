# DNNProv

## Overview

DNNProv is a provenance service designed for supporting online hyperparameter analysis in Deep Neural Networks (DNNs). DNNProv integrates traditional retrospective provenance data (r-prov) with typical DNN software data, e.g. hyperparameters, DNN architecture attributes, etc. This solution provides an API that allows for users to develop their DNN-based workflows using any software library for deep learning while being able to analyze online captured provenance data.

DNNProv is developed on top of [DfAnalyzer](https://gitlab.com/ssvitor/dataflow_analyzer) provenance services. It uses the columnar DBMS MonetDB to support online provenance data analysis and to generate W3C PROV compliant documents.

## Software requirements

The following softwares have to be configured/installed for running a DNN training that collects provenance with DNNProv.

* [Java](https://java.com/pt-BR/)
* [MonetDB](http://www.monetdb.org/Documentation/UserGuide/Tutorial)
* [DfAnalyzer](https://github.com/dbpina/keras-prov/tree/main/DfAnalyzer)
* [dfa-lib-python](https://github.com/dbpina/keras-prov/tree/main/dfa-lib-python/) 

## Installation

<!---### RESTful services -->


###  Python library: dfa-lib-python

The DfAnalyzer library for the programming language Python can be built with the following command lines:

```

cd dfa-lib-python
python setup.py install

```

## RESTful services initialization

DNNProv depends on the initialization of DfAnalyzer and the DBMS MonetDB.

Instructions for this step can also be found at [GitLab](https://gitlab.com/ssvitor/dataflow_analyzer). The project DfAnalyzer contains web applications and RESTful services provided by the tool. 

The following components are present in this project: Dataflow Viewer (DfViewer), Query Interface (QI), and Query Dashboard (QP). We provide a compressed file of our MonetDB database (to DfAnalyzer) for a local execution of the project DfAnalyzer. Therefore, users only need to run the script start-dfanalyzer.sh at the path DfAnalyzer. We assume the execution of these steps with a Unix-based operating system, as follows:

```

cd DfAnalyzer
./start-dfanalyzer.sh

```

## How to run DNN applications

The DNNProv has a few predefined hyperparameters (e.g. optimizer, learning rate, number of epochs, number of layer, etc.) and metrics (e.g. loss, accuracy, elapsed time) to be captured. In the case that these hyperparameter ans metrics are enough, the user have to set the attribute “hyperparameters” as True, and the library will take care of it. It's importante to set a tag to identify the workflow and associate to the provenance data, e.g. hyperparameters. This method captures provenance data as the the deep learning workflow executes and sends them to the provenance database managed by MonetDB. As the data reaches the database, it can be analyzed through the Dataflow Viewer (DfViewer), Query Interface (QI), and Query Dashboard (QP). The data received by the provenance method are defined by the user in the source code of the DNN application, as follows:

```
df = Dataflow(dataflow_tag, hyperparameters=True)
df.save()
```

To capture the retrospective provenance, the user should add the following code:

```
tf1_input = DataSet("iTrainingModel", [Element([opt.get_config()['name'], opt.get_config()['learning_rate'], epochs, len(model.layers)])])
t1.add_dataset(tf1_input)
t1.begin() 

## Data manipulation

tf1_output = DataSet("oTrainingModel", [Element([datetime.now().strftime('%Y-%m-%d %H:%M:%S'), elapsed_time, loss, accuracy, val_loss, val_accuracy, epoch])])
t1.add_dataset(tf1_output)
if(epoch==final_epoch):
	t1.end()
else:
	t1.save()    
```

In case there is an adaptation of the hyperparameters during training (e.g., an update of the learning rate), that is, the use of methods such as LearningRateScheduler offered by Keras, the hyperparameter’s values are updated, therefore, the adaptation should be registered for further analysis. To capture these data, the user should add code for this specific transformation.

## Example

The path `Example` shows how to use Keras-Prov. To run it, the user just needs to run the python command, as follows: 

```
python alexnet_dnnprov.py
```

To add new parameters, hyperparameter or metrics to be captured and stored, the user needs to specify the new transformation. For example, if they want to capture data related to the DNN architecture like a dense block (growth rate and number of layers in the dense block), the specification has to be added before the model.fit command on user's training code and should be like:

```
df = model.get_dataflow()

tf_denseb = Transformation("DenseBlock")
tf_denseb_input = Set("iDenseBlock", SetType.INPUT, 
    [Attribute("growth_rate", AttributeType.NUMERIC), 
    Attribute("layers_db", AttributeType.NUMERIC)])
tf_denseb_output = Set("oDenseBlock", SetType.OUTPUT, 
    [Attribute("output", AttributeType.TEXT)])
tf_denseb.set_sets([tf_denseb_input, tf_denseb_output])
df.add_transformation(tf_denseb) 
```

The second step is the moment when the user must instrument the code to capture the parameter value. For example:

```
t_denseb = Task(identifier=4, dataflow_tag, "DenseBlock")
##Data manipulation, example:
growth_rate = 1
layers_db = 33
t_denseb_input = DataSet("iExtrairNumeros", [Element([growth_rate, layers_db])])
t_denseb.add_dataset(t_denseb_input)
t_denseb.begin()
##Data manipulation, example:
t_denseb_output= DataSet("oExtrairNumeros", [Element([output])])
t_denseb.add_dataset(t_denseb_output)
t_denseb.end()
```

Both steps, specification of the transformation and the activity definition, follow the definitions of [dfa-lib-python](http://monografias.poli.ufrj.br/monografias/monopoli10026387.pdf) for DfAnalyzer.


## Presentation Video

To watch the video, please, click [here](https://www.youtube.com/watch?v=QOZY2CQfXJ8).

## Publications

* [Pina, D. B., Neves, L., Paes, A., de Oliveira, D., & Mattoso, M. (2019, November). Análise de hiperparâmetros em aplicações de aprendizado profundo por meio de dados de proveniência. In Anais do XXXIV Simpósio Brasileiro de Banco de Dados (pp. 223-228). SBC.](https://sol.sbc.org.br/index.php/sbbd/article/view/8827)

* [Pina, D., Kunstmann, L., de Oliveira, D., Valduriez, P., & Mattoso, M. (2020, September). Uma abordagem para coleta e análise de dados de configurações em redes neurais profundas. In 35ª Simpósio Brasileiro de Banco de Dados (SBBD) (pp. 1-6).](https://hal-lirmm.ccsd.cnrs.fr/lirmm-02969506/)

* [Pina, D., Kunstmann, L., de Oliveira, D., Valduriez, P., & Mattoso, M. (2020). Provenance Supporting Hyperparameter Analysis in Deep Neural Networks. In Provenance and Annotation of Data and Processes (pp. 20-38). Springer, Cham.](https://link.springer.com/chapter/10.1007/978-3-030-80960-7_2)

* [Pina, D., Neves, L., de Oliveira, D., & Mattoso, M. (2021, October). Captura Automática de Dados de Proveniência de Experimentos de Aprendizado de Máquina com Keras-Prov. In Anais Estendidos do XXXVI Simpósio Brasileiro de Bancos de Dados (pp. 69-74). SBC.](https://sol.sbc.org.br/index.php/sbbd_estendido/article/view/18165)
