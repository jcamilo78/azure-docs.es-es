---
title: Creación de experimentos de ML automatizados
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo definir orígenes de datos, procesos y valores de configuración para los experimentos de aprendizaje automático automatizado.
author: cartacioS
ms.author: sacartac
ms.reviewer: nibaccam
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 09/29/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python,contperf-fy21q1, automl
ms.openlocfilehash: 9021d933e3808867ec784ad3c6d0f8810d608ea3
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/20/2021
ms.locfileid: "98600061"
---
# <a name="configure-automated-ml-experiments-in-python"></a>Configuración de experimentos de ML automatizado en Python


En esta guía, aprenderá a definir diversos valores de configuración de los experimentos de aprendizaje automático automatizado con el [SDK de Azure Machine Learning](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py). El aprendizaje automático automatizado elige un algoritmo e hiperparámetros, y genera un modelo listo para la implementación. Se pueden usar varias opciones para configurar experimentos de aprendizaje automático automatizado.

Para ver ejemplos de experimentos de aprendizaje automático automatizado, consulte [Tutorial: Train a classification model with automated machine learning](tutorial-auto-train-models.md) (Tutorial: Entrenamiento de un modelo de clasificación con aprendizaje automático automatizado) o [Entrenamiento de modelos con aprendizaje automático automatizado en la nube mediante Azure Machine Learning](how-to-auto-train-remote.md).

Opciones de configuración disponibles en el aprendizaje automático automatizado:

* Selección del tipo de experimento: Clasificación, regresión o predicción de las series temporales
* Origen de datos, formatos y datos de captura
* Elección el destino del proceso: local o remoto
* Configuración de experimentos de aprendizaje automático automatizado
* Ejecución de un experimento de aprendizaje automático automatizado
* Explorar las métricas del modelo
* Registrar e implementar el modelo

Si prefiere una experiencia sin código, también puede [crear experimentos de aprendizaje automático automatizados en Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md).

## <a name="prerequisites"></a>Requisitos previos

Para realizar este artículo, necesitará lo siguiente 
* Un área de trabajo de Azure Machine Learning. Para crear el área de trabajo, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

* El SDK de Azure Machine Learning para Python instalado.
    Para instalar el SDK, puede: 
    * Crear una instancia de proceso, que instala automáticamente el SDK y está preconfigurada para flujos de trabajo de aprendizaje automático. Consulte [Creación y administración de una instancia de proceso de Azure Machine Learning](how-to-create-manage-compute-instance.md) para obtener más información. 

    * [Instale el paquete `automl`](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/README.md#setup-using-a-local-conda-environment), que incluye la [instalación predeterminada](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py#default-install) del SDK.

## <a name="select-your-experiment-type"></a>Seleccione el tipo de experimento

Antes de comenzar el experimento, debe determinar el tipo de problema de aprendizaje automático que va a resolver. El aprendizaje automático automatizado admite tipos de tareas de `classification`, `regression` y `forecasting`. Más información sobre los [tipos de tareas](concept-automated-ml.md#when-to-use-automl-classify-regression--forecast).

En el código siguiente se usa el parámetro `task` en el constructor `AutoMLConfig` para especificar el tipo de experimento como `classification`.

```python
from azureml.train.automl import AutoMLConfig

# task can be one of classification, regression, forecasting
automl_config = AutoMLConfig(task = "classification")
```

## <a name="data-source-and-format"></a>Formato y origen de datos

El aprendizaje automático automatizado es compatible con los datos que residen en el escritorio local o en la nube, como Azure Blob Storage. Los datos se pueden leer en **DataFrame de Pandas** o en **TabularDataset de Azure Machine Learning**. [Más información sobre los conjuntos de datos](how-to-create-register-datasets.md).

Requisitos para los datos de entrenamiento en el aprendizaje automático:
- Los datos deben estar en formato tabular.
- El valor que se va a predecir, la columna de destino, debe estar en los datos.

**En el caso de los experimentos remotos**, los datos de aprendizaje deben ser accesibles desde el proceso remoto. AutoML solo acepta [la clase TabularDataset de Azure Machine Learning](/python/api/azureml-core/azureml.data.tabulardataset?preserve-view=true&view=azure-ml-py) al trabajar en un proceso remoto. 

Los conjuntos de datos de Azure Machine Learning exponen la funcionalidad para:

* Transferir datos fácilmente de archivos estáticos u orígenes de dirección URL a su área de trabajo.
* Poner sus datos a disposición de los scripts de entrenamiento al ejecutarse en recursos de proceso en la nube. Consulte [Entrenamiento con conjuntos de datos](how-to-train-with-datasets.md#mount-files-to-remote-compute-targets) para obtener un ejemplo del uso de la clase `Dataset` para montar datos en el destino de proceso remoto.

El código siguiente crea un objeto TabularDataset a partir de una dirección URL web. Consulte [Creación de un objeto TabularDataset](how-to-create-register-datasets.md#create-a-tabulardataset) para ver ejemplos de código sobre cómo crear conjuntos de datos desde otros orígenes, como archivos locales y almacenes de datos.

```python
from azureml.core.dataset import Dataset
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"
dataset = Dataset.Tabular.from_delimited_files(data)
  ```
**En el caso de los experimentos de proceso locales**, se recomiendan dataframes de Pandas para acelerar los tiempos de procesamiento.

  ```python
  import pandas as pd
  from sklearn.model_selection import train_test_split

  df = pd.read_csv("your-local-file.csv")
  train_data, test_data = train_test_split(df, test_size=0.1, random_state=42)
  label = "label-col-name"
  ```

## <a name="training-validation-and-test-data"></a>Datos de entrenamiento, validación y prueba

Puede especificar **datos del entrenamiento y conjuntos de datos de validación**  independientes directamente en el constructor `AutoMLConfig`. Más información sobre [cómo configurar las divisiones de datos y la validación cruzada](how-to-configure-cross-validation-data-splits.md) para los experimentos de AutoML. 

Si no especifica explícitamente un parámetro `validation_data` o `n_cross_validation`, AutoML aplica las técnicas predeterminadas para determinar cómo se realiza la validación. Esta determinación depende del número de filas del conjunto de datos asignadas a su parámetro `training_data`. 

|Tamaño&nbsp;de datos de&nbsp;entrenamiento| Técnica de validación |
|---|-----|
|**Mayor&nbsp;que&nbsp;20 000&nbsp;filas**| Se aplica la división de datos de entrenamiento o validación. El valor predeterminado consiste en usar el 10 % del conjunto de datos de entrenamiento inicial como conjunto de validación. A su vez, ese conjunto de validación se usa para calcular las métricas.
|**Menor&nbsp;que&nbsp;20 000&nbsp;filas**| Se aplica el enfoque de validación cruzada. El número predeterminado de iteraciones depende del número de filas. <br> **Si el conjunto de datos tiene menos de 1000 filas**, se usan diez iteraciones. <br> **Si hay entre 1000 y 20 000 filas**, se usan tres iteraciones.

En este momento, debe proporcionar sus propios **datos de prueba** para la evaluación del modelo. Para obtener un ejemplo de código sobre cómo aportar sus propios datos de prueba para la evaluación del modelo, consulte la sección de **pruebas** de [este cuaderno de Jupyter](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-credit-card-fraud/auto-ml-classification-credit-card-fraud.ipynb).

## <a name="compute-to-run-experiment"></a>Proceso para ejecutar el experimento

A continuación, determine dónde se va a entrenar el modelo. Un experimento de entrenamiento de aprendizaje automático automatizado se puede ejecutar en las opciones de proceso siguientes. Obtenga información sobre las [ventajas y desventajas de las opciones de proceso locales y remotas](concept-automated-ml.md#local-remote). 

* La máquina **local**, como un escritorio local o un equipo portátil: generalmente, cuando haya un pequeño conjunto de datos y siga en la fase de exploración. Consulte [este cuaderno](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/local-run-classification-credit-card-fraud/auto-ml-classification-credit-card-fraud-local.ipynb) para obtener un ejemplo de proceso local. 
 
* Una máquina [remota](concept-compute-target.md#amlcompute) en la nube: **Azure Machine Learning Managed Compute** es un servicio administrado que permite entrenar modelos de aprendizaje automático en clústeres de máquinas virtuales de Azure. 

    Consulte [este cuaderno](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features/auto-ml-classification-bank-marketing-all-features.ipynb) para ver un ejemplo remoto con una instancia de Azure Machine Learning Managed Compute. 

* Un **clúster de Azure Databricks** en su suscripción de Azure. Puede encontrar más detalles en [Configuración del clúster de Azure Databricks para ML automatizado](how-to-configure-databricks-automl-environment.md). Consulte este [sitio de GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/azure-databricks/automl) para ver ejemplos de cuadernos con Azure Databricks.

<a name='configure-experiment'></a>

## <a name="configure-your-experiment-settings"></a>Establecer la configuración de experimento

Se pueden usar varias opciones para configurar el experimento de aprendizaje automático automatizado. Estos parámetros se establecen al crear una instancia un objeto `AutoMLConfig`. Consulte la [clase AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig) para obtener una lista completa de parámetros.

Estos son algunos ejemplos:

1. Experimento de clasificación con AUC ponderado como métrica principal con los minutos de tiempo de espera del experimento establecidos en 30 minutos y 2 iteraciones de validación cruzada.

   ```python
       automl_classifier=AutoMLConfig(task='classification',
                                      primary_metric='AUC_weighted',
                                      experiment_timeout_minutes=30,
                                      blocked_models=['XGBoostClassifier'],
                                      training_data=train_data,
                                      label_column_name=label,
                                      n_cross_validations=2)
   ```
1. A continuación se incluye un ejemplo de un conjunto de experimentos de regresión que finaliza después de 60 minutos con 5 iteraciones de validación cruzada.

   ```python
      automl_regressor = AutoMLConfig(task='regression',
                                      experiment_timeout_minutes=60,
                                      allowed_models=['KNN'],
                                      primary_metric='r2_score',
                                      training_data=train_data,
                                      label_column_name=label,
                                      n_cross_validations=5)
   ```


1. Las tareas de previsión requieren una configuración adicional; vea el artículo [Entrenamiento automático de un modelo de previsión de series temporales](how-to-auto-train-forecast.md) para obtener más detalles. 

    ```python
    time_series_settings = {
        'time_column_name': time_column_name,
        'time_series_id_column_names': time_series_id_column_names,
        'forecast_horizon': n_test_periods
    }
    
    automl_config = AutoMLConfig(task = 'forecasting',
                                 debug_log='automl_oj_sales_errors.log',
                                 primary_metric='normalized_root_mean_squared_error',
                                 experiment_timeout_minutes=20,
                                 training_data=train_data,
                                 label_column_name=label,
                                 n_cross_validations=5,
                                 path=project_folder,
                                 verbosity=logging.INFO,
                                 **time_series_settings)
    ```
    
### <a name="supported-models"></a>Modelos admitidos

El aprendizaje automático automatizado prueba diferentes algoritmos y modelos durante el proceso de optimización y automatización. Como usuario, no hay ninguna necesidad de especificar el algoritmo. 

Los tres valores de parámetro `task` diferentes determinan la lista de algoritmos o modelos que se aplicará. Use los parámetros `allowed_models` o `blocked_models` para modificar aún más las iteraciones con los modelos disponibles para incluir o excluir. 

En la tabla siguiente se resumen los modelos admitidos por tipo de tarea. 

> [!NOTE]
> Si planea exportar los modelos creados mediante ML a un [modelo de ONNX](concept-onnx.md), solo los algoritmos que se indican con un * se pueden convertir al formato ONNX. Más información sobre la [conversión de modelos a ONNX](concept-automated-ml.md#use-with-onnx). <br> <br> Tenga en cuenta también que, en este momento, ONNX solo admite tareas de clasificación y regresión. 

clasificación | Regresión | Previsión de series temporales
|-- |-- |--
[Regresión logística](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression)* | [Red elástica](https://scikit-learn.org/stable/modules/linear_model.html#elastic-net)* | [Red elástica](https://scikit-learn.org/stable/modules/linear_model.html#elastic-net)
[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)* |[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)*|[Light GBM](https://lightgbm.readthedocs.io/en/latest/index.html)
[Potenciación del gradiente](https://scikit-learn.org/stable/modules/ensemble.html#classification)* |[Potenciación del gradiente](https://scikit-learn.org/stable/modules/ensemble.html#regression)* |[Potenciación del gradiente](https://scikit-learn.org/stable/modules/ensemble.html#regression)
[Árbol de decisión](https://scikit-learn.org/stable/modules/tree.html#decision-trees)* |[Árbol de decisión](https://scikit-learn.org/stable/modules/tree.html#regression)* |[Árbol de decisión](https://scikit-learn.org/stable/modules/tree.html#regression)
[K Vecinos más próximos](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)* |[K Vecinos más próximos](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)* |[K Vecinos más próximos](https://scikit-learn.org/stable/modules/neighbors.html#nearest-neighbors-regression)
[SVC lineal](https://scikit-learn.org/stable/modules/svm.html#classification)* |[Lazo LARS](https://scikit-learn.org/stable/modules/linear_model.html#lars-lasso)* |[Lazo LARS](https://scikit-learn.org/stable/modules/linear_model.html#lars-lasso)
[Clasificación de vectores de soporte (SVC)](https://scikit-learn.org/stable/modules/svm.html#classification)* |[Descenso de gradiente estocástico (SGD)](https://scikit-learn.org/stable/modules/sgd.html#regression)* |[Descenso de gradiente estocástico (SGD)](https://scikit-learn.org/stable/modules/sgd.html#regression)
[Bosque aleatorio](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)* |[Bosque aleatorio](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)* |[Bosque aleatorio](https://scikit-learn.org/stable/modules/ensemble.html#random-forests)
[Árboles extremadamente aleatorios](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)* |[Árboles extremadamente aleatorios](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)* |[Árboles extremadamente aleatorios](https://scikit-learn.org/stable/modules/ensemble.html#extremely-randomized-trees)
[Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)* |[Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)* | [Xgboost](https://xgboost.readthedocs.io/en/latest/parameter.html)
[Clasificador de perceptrón promedio](/python/api/nimbusml/nimbusml.linear_model.averagedperceptronbinaryclassifier?preserve-view=true&view=nimbusml-py-latest)|[Regresor descendente de gradiente en línea](/python/api/nimbusml/nimbusml.linear_model.onlinegradientdescentregressor?preserve-view=true&view=nimbusml-py-latest) |[Auto-ARIMA](https://www.alkaline-ml.com/pmdarima/modules/generated/pmdarima.arima.auto_arima.html#pmdarima.arima.auto_arima)
[Naive Bayes](https://scikit-learn.org/stable/modules/naive_bayes.html#bernoulli-naive-bayes)* |[Regresor lineal rápida](/python/api/nimbusml/nimbusml.linear_model.fastlinearregressor?preserve-view=true&view=nimbusml-py-latest)|[Prophet](https://facebook.github.io/prophet/docs/quick_start.html)
[Descenso de gradiente estocástico (SGD)](https://scikit-learn.org/stable/modules/sgd.html#sgd)* ||ForecastTCN
|[Clasificador SVM lineal](/python/api/nimbusml/nimbusml.linear_model.linearsvmbinaryclassifier?preserve-view=true&view=nimbusml-py-latest)*||

### <a name="primary-metric"></a>Métrica principal
El parámetro `primary metric` determina la métrica que se utilizará durante el entrenamiento del modelo para la optimización. Las métricas disponibles que puede seleccionar vienen determinadas por el tipo de tarea que elige y en la siguiente tabla se muestran métricas principales válidas para cada tipo de tarea.

Obtenga información acerca de las definiciones específicas de estas métricas en [Descripción de los resultados de aprendizaje automático automatizado](how-to-understand-automated-ml.md).

|clasificación | Regresión | Previsión de series temporales
|--|--|--
|accuracy| spearman_correlation | spearman_correlation
|AUC_weighted | normalized_root_mean_squared_error | normalized_root_mean_squared_error
|average_precision_score_weighted | r2_score | r2_score
|norm_macro_recall | normalized_mean_absolute_error | normalized_mean_absolute_error
|precision_score_weighted |

### <a name="data-featurization"></a>Caracterización de datos

En cada experimento de aprendizaje automático automatizado, los datos se escalan y se normalizan automáticamente para ayudar a *determinados* algoritmos que dependen de características que se encuentran en diferentes escalas. Este ajuste de escala y normalización se conoce como caracterización. Consulte [Caracterización en aprendizaje automático automatizado](how-to-configure-auto-features.md#) para obtener más información y ejemplos de código. 

Al configurar los experimentos en el objeto `AutoMLConfig`, puede habilitar o deshabilitar la opción de configuración `featurization`. En la tabla siguiente se muestra la configuración aceptada para la caracterización del objeto [AutoMLConfig](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig). 

|Configuración de la caracterización | Descripción |
| ------------- | ------------- |
|`"featurization": 'auto'`| Indica que, como parte del preprocesamiento, los [pasos de caracterización y protección](how-to-configure-auto-features.md#featurization) se realizan automáticamente. **Valor predeterminado**.|
|`"featurization": 'off'`| Indica que el paso de caracterización no se debe realizar de forma automática.|
|`"featurization":`&nbsp;`'FeaturizationConfig'`| Indica que se debe usar un paso personalizado de caracterización. [Aprenda a personalizar la caracterización](how-to-configure-auto-features.md#customize-featurization).|

> [!NOTE]
> Los pasos de la caracterización del aprendizaje automático automatizado (normalización de características, control de los datos que faltan, conversión de valores de texto a numéricos, etc.) se convierten en parte del modelo subyacente. Cuando se usa el modelo para realizar predicciones, se aplican automáticamente a los datos de entrada los mismos pasos de caracterización que se aplican durante el entrenamiento.

<a name="ensemble"></a>

### <a name="ensemble-configuration"></a> Configuración de conjuntos

Los modelos de conjunto están habilitados de forma predeterminada y aparecen como las iteraciones de ejecución finales en una ejecución de AutoML. Actualmente se admiten **VotingEnsemble** y **StackEnsemble**. 

La votación se implementa como un voto flexible mediante promedios ponderados. La implementación de apilamiento usa una implementación de dos niveles, donde el primer nivel tiene los mismos modelos que el conjunto de votación y el segundo modelo de nivel se usa para encontrar la combinación óptima de los modelos del primer nivel. 

Si usa modelos de ONNX **o** tiene habilitada la explicación del modelo, el apilamiento se deshabilita y solo se utiliza la votación.

El entrenamiento de conjunto se puede deshabilitar mediante los parámetros booleanos `enable_voting_ensemble` y `enable_stack_ensemble`.

```python
automl_classifier = AutoMLConfig(
        task='classification',
        primary_metric='AUC_weighted',
        experiment_timeout_minutes=30,
        training_data=data_train,
        label_column_name=label,
        n_cross_validations=5,
        enable_voting_ensemble=False,
        enable_stack_ensemble=False
        )
```

Hay varios argumentos predeterminados que se pueden proporcionar como `kwargs` en un objeto `AutoMLConfig` a fin de modificar el comportamiento predeterminado del conjunto.

> [!IMPORTANT]
>  Los parámetros siguientes no son parámetros explícitos de la clase AutoMLConfig. 

* `ensemble_download_models_timeout_sec`: Durante la generación de los modelos **VotingEnsemble** y **StackEnsemble**, se descargan varios modelos ajustados de las ejecuciones secundarias anteriores. Si detecta el error `AutoMLEnsembleException: Could not find any models for running ensembling`, es posible que tenga que proporcionar más tiempo para que se descarguen los modelos. El valor predeterminado es de 300 segundos para descargar estos modelos en paralelo y no hay límite máximo de tiempo de expiración. Configure este parámetro con un valor superior a 300 segundos, si se necesita más tiempo. 

  > [!NOTE]
  >  Si se alcanza el tiempo de expiración y hay modelos descargados, el ensamblado continúa con todos los modelos que ha descargado. No es necesario que se descarguen todos los modelos para finalizar en ese tiempo de expiración.

Los parámetros siguientes solo se aplican a los modelos **StackEnsemble**: 

* `stack_meta_learner_type`: el metaaprendizaje es un modelo entrenado en la salida de los modelos heterogéneos individuales. Los metaaprendizajes predeterminados son `LogisticRegression` para las tareas de clasificación (o `LogisticRegressionCV` si está habilitada la validación cruzada) y `ElasticNet` para las tareas de regresión y predicción (o `ElasticNetCV` si está habilitada la validación cruzada). Este parámetro puede ser una de las cadenas siguientes: `LogisticRegression`, `LogisticRegressionCV`, `LightGBMClassifier`, `ElasticNet`, `ElasticNetCV`, `LightGBMRegressor` o `LinearRegression`.

* `stack_meta_learner_train_percentage`: especifica la proporción del conjunto de entrenamiento (al elegir entrenar y el tipo de validación de entrenamiento) que se va a reservar para entrenar el metaaprendizaje. El valor predeterminado es `0.2`. 

* `stack_meta_learner_kwargs`: parámetros opcionales que se van a pasar al inicializador del metaaprendizaje. Estos parámetros y tipos de parámetro reflejan los parámetros y tipos de parámetro del constructor del modelo correspondiente y se reenvían a dicho constructor.

En el código siguiente se muestra un ejemplo de cómo especificar el comportamiento del conjunto personalizado en un objeto `AutoMLConfig`.

```python
ensemble_settings = {
    "ensemble_download_models_timeout_sec": 600
    "stack_meta_learner_type": "LogisticRegressionCV",
    "stack_meta_learner_train_percentage": 0.3,
    "stack_meta_learner_kwargs": {
        "refit": True,
        "fit_intercept": False,
        "class_weight": "balanced",
        "multi_class": "auto",
        "n_jobs": -1
    }
}

automl_classifier = AutoMLConfig(
        task='classification',
        primary_metric='AUC_weighted',
        experiment_timeout_minutes=30,
        training_data=train_data,
        label_column_name=label,
        n_cross_validations=5,
        **ensemble_settings
        )
```

<a name="exit"></a> 

### <a name="exit-criteria"></a>Exit criteria (Criterios de salida)

Hay algunas opciones que puede definir en el AutoMLConfig para finalizar el experimento.

|Criterios| description
|----|----
Sin criterio | Si no se define ningún parámetro de salida, el experimento continúa hasta que no se realice ningún progreso adicional en la métrica principal.
Transcurrido&nbsp;un&nbsp;tiempo&nbsp;determinado&nbsp;| Use `experiment_timeout_minutes` en la configuración para definir el tiempo, en minutos, en que el experimento debe seguir ejecutándose. <br><br> Para ayudar a evitar errores de tiempo de espera del experimento, hay un mínimo de 15 minutos o 60 minutos si el tamaño de fila por columna es superior a 10 millones.
Una&nbsp;vez&nbsp;alcanzada&nbsp;una&nbsp;puntuación| El uso de `experiment_exit_score` completará el experimento una vez alcanzada una puntuación de métrica principal especificada.

## <a name="run-experiment"></a>Ejecutar experimento

Para una instancia de ML automatizado, se crea un objeto `Experiment`, que es un objeto con nombre en un objeto `Workspace` que se usa para ejecutar experimentos.

```python
from azureml.core.experiment import Experiment

ws = Workspace.from_config()

# Choose a name for the experiment and specify the project folder.
experiment_name = 'Tutorial-automl'
project_folder = './sample_projects/automl-classification'

experiment = Experiment(ws, experiment_name)
```

Envíe el experimento para ejecutar y generar un modelo. Pase `AutoMLConfig` al método `submit` para generar el modelo.

```python
run = experiment.submit(automl_config, show_output=True)
```

>[!NOTE]
>Las dependencias se instalan por primera vez en una máquina nueva.  El resultado puede tardar hasta 10 minutos en mostrarse.
>Establecer `show_output` en `True` genera un resultado que se muestra en la consola.

### <a name="multiple-child-runs-on-clusters"></a>Varias ejecuciones secundarias en clústeres

Las ejecuciones secundarias de experimentos de aprendizaje automático automatizado se pueden realizar en un clúster que ya está ejecutando otro experimento. Sin embargo, el tiempo depende del número de nodos que tenga el clúster y de si esos nodos están disponibles para ejecutar un experimento diferente.

Cada nodo del clúster actúa como una máquina virtual (VM) individual que puede realizar una sola ejecución de entrenamiento; en el caso del aprendizaje automático automatizado, esto significa una ejecución secundaria. Si todos los nodos están ocupados, el nuevo experimento se pone en cola. Sin embargo, si hay nodos libres, el nuevo experimento ejecutará las ejecuciones secundarias de aprendizaje automático automatizado en paralelo en los nodos o máquinas virtuales disponibles.

Para ayudar a administrar las ejecuciones secundarias y cuándo se pueden realizar, se recomienda crear un clúster dedicado por experimento y hacer coincidir el número de `max_concurrent_iterations` del experimento con el número de nodos del clúster. De esta manera, se usan todos los nodos del clúster al mismo tiempo con el número de ejecuciones o iteraciones secundarias simultáneas que se desee.

Configure `max_concurrent_iterations` en el objeto `AutoMLConfig`. Si no se configura, solo se permite de forma predeterminada una ejecución o iteración secundaria simultánea en cada experimento.  

## <a name="explore-models-and-metrics"></a>Exploración de modelos y métricas

Puede ver los resultados del entrenamiento en un widget o en línea si se encuentra en un bloc de notas. Vea [Seguimiento y evaluación de modelos](how-to-monitor-view-training-logs.md#monitor-automated-machine-learning-runs) para obtener más información.

Consulte [Evaluación de los resultados de los experimentos de aprendizaje automático automatizado](how-to-understand-automated-ml.md) para ver definiciones y ejemplos de los gráficos de rendimiento y las métricas proporcionadas para cada ejecución. 

Para obtener un resumen de la caracterización y comprender las características que se agregaron a un modelo determinado, consulte [Transparencia de caracterización](how-to-configure-auto-features.md#featurization-transparency). 

> [!NOTE]
> Los algoritmos que el aprendizaje automático automatizado emplea llevan inherente la aleatoriedad, que puede provocar una ligera variación en la puntuación de las métricas finales del modelo recomendado, como la precisión. El aprendizaje automático automatizado también realiza operaciones en datos, como la división de la prueba de entrenamiento, la división de la validación de entrenamiento o la validación cruzada cuando es necesario. Por lo tanto, si ejecuta un experimento con las mismas opciones de configuración y métricas principales varias veces, es probable que vea una variación en las puntuaciones de las métricas finales de los experimentos debido a estos factores. 

## <a name="register-and-deploy-models"></a>Registro e implementación de modelos

Para más información sobre cómo descargar o registrar un modelo para la implementación en un servicio Web, consulte [cómo y dónde implementar un modelo](how-to-deploy-and-where.md).

<a name="explain"></a>

## <a name="model-interpretability"></a>Interoperabilidad del modelo

La interpretabilidad del modelo permite comprender por qué los modelos hacen predicciones y los valores de importancia de las características subyacentes. El SDK incluye varios paquetes para habilitar las características de interpretabilidad del modelo, tanto en el momento del entrenamiento como en el de inferencia, para los modelos implementados y locales.

Vea el [procedimiento](how-to-machine-learning-interpretability-automl.md) de los ejemplos de código sobre cómo habilitar características de interpretabilidad específicamente dentro de experimentos de aprendizaje automático automatizados.

Para información general sobre cómo se pueden habilitar las explicaciones del modelo y la importancia de las características en otras áreas del SDK fuera del aprendizaje automático automatizado, consulte el artículo de [conceptos](how-to-machine-learning-interpretability.md) sobre la interpretabilidad.

> [!NOTE]
> El modelo ForecastTCN no es compatible en la actualidad con el cliente de explicación. Este modelo no devolverá un panel de explicación si se devuelve como el mejor modelo, y no admite ejecuciones de explicación a petición.

## <a name="troubleshooting"></a>Solución de problemas

* **La actualización reciente de las dependencias de `AutoML` a versiones más recientes anulará la compatibilidad**:  A partir de la versión 1.13.0 del SDK, los modelos no se cargan en los SDK anteriores, ya que existe incompatibilidad entre las versiones anteriores que se anclaron en los paquetes anteriores y las versiones más recientes que se anclan ahora. Verá un error similar a alguno de estos:
  * Módulo no encontrado: por ejemplo, `No module named 'sklearn.decomposition._truncated_svd`.
  * Errores de importación: por ejemplo, `ImportError: cannot import name 'RollingOriginValidator'`.
  * Errores de atributos: por ejemplo, `AttributeError: 'SimpleImputer' object has no attribute 'add_indicator`
  
  Para solucionar este problema, realice uno de los dos pasos siguientes, en función de la versión de entrenamiento del SDK de `AutoML`:
    * Si la versión de entrenamiento del SDK de `AutoML` es posterior a la 1.13.0, se necesitan `pandas == 0.25.1` y `sckit-learn==0.22.1`. Si hay una discrepancia de versiones, actualice scikit-learn o pandas a la versión correcta, como se muestra a continuación:
      
      ```bash
         pip install --upgrade pandas==0.25.1
         pip install --upgrade scikit-learn==0.22.1
      ```
      
    * Si la versión de entrenamiento del SDK de `AutoML` es la 1.12.0, o anterior, se necesitan `pandas == 0.23.4` y `sckit-learn==0.20.3`. Si hay una discrepancia de versiones, actualice scikit-learn o pandas a la versión correcta como se muestra a continuación:
  
      ```bash
        pip install --upgrade pandas==0.23.4
        pip install --upgrade scikit-learn==0.20.3
      ```

* **Implementación con errores**: En el caso de la versión 1.18.0 u otras anteriores del SDK, puede producirse el siguiente error en la imagen base creada para la implementación: "ImportError: no se puede importar el nombre `cached_property` de `werkzeug`". 

  Para solucionar el problema, haga lo siguiente:
  1. Descargue los archivos del modelo.
  2. Descompresión del paquete
  3. Implementación mediante los recursos sin comprimir

* **La puntuación R2 de la previsión siempre es cero**: este problema surge si los datos de entrenamiento proporcionados tienen una serie temporal que contiene el mismo valor para los últimos puntos de datos `n_cv_splits` + `forecasting_horizon`. Si se espera este patrón en la serie temporal, puede cambiar la métrica principal a la raíz del error cuadrático medio normalizado.
 
* **TensorFlow**: A partir de la versión 1.5.0 del SDK, el aprendizaje automático automatizado no instala los modelos de TensorFlow de forma predeterminada. Para instalar TensorFlow y usarlo con los experimentos de aprendizaje automático automatizado, instale TensorFlow==1.12.0 mediante CondaDependecies. 
 
   ```python
   from azureml.core.runconfig import RunConfiguration
   from azureml.core.conda_dependencies import CondaDependencies
   run_config = RunConfiguration()
   run_config.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['tensorflow==1.12.0'])
  ```

* **Gráficos de experimento**: Los gráficos de clasificación binaria (precisión-retirada, ROC, curva de ganancia, etc.) que se muestran en las iteraciones de experimentos de ML automatizados no se representan correctamente en la interfaz de usuario desde el 12/04. Los trazados de los gráficos actualmente muestran resultados inversos, donde los modelos con mejor rendimiento se muestran con resultados inferiores. Se está investigando una resolución.

* **Cancelación de Databricks de una ejecución de aprendizaje automático automatizado**: Al usar las funcionalidades de aprendizaje automático automatizado en Azure Databricks, para cancelar una ejecución e iniciar una nueva ejecución de un experimento, reinicie el clúster de Azure Databricks.

* **> 10 iteraciones de Databricks para aprendizaje automático automatizado**: En la configuración del aprendizaje automático automatizado, si tiene más de 10 iteraciones, establezca `show_output` en `False` cuando envíe la ejecución.

* **Widget de Databricks para el SDK de Azure Machine Learning y aprendizaje automático automatizado**: El widget del SDK de Azure Machine Learning no se admite en un cuaderno de Databricks porque los cuadernos no pueden analizar los widgets HTML. Para ver el widget en el portal, use este código de Python en la celda del cuaderno de Azure Databricks:

    ```
    displayHTML("<a href={} target='_blank'>Azure Portal: {}</a>".format(local_run.get_portal_url(), local_run.id))
    ```
* **Error de automl_setup**: 
    * En Windows, ejecute automl_setup desde un símbolo del sistema de Anaconda. Use este vínculo para [instalar Miniconda](https://docs.conda.io/en/latest/miniconda.html).
    * Asegúrese de que CONDA de 64 bits está instalado, en lugar de 32 bits mediante la ejecución del comando `conda info`. `platform` debe ser `win-64` para Windows o `osx-64` para Mac.
    * Asegúrese de está instalado Conda 4.4.10, o una versión posterior. Puede comprobar la versión con el comando `conda -V`. Si tiene instalada una versión anterior, puede actualizarla mediante el comando `conda update conda`.
    * Linux: `gcc: error trying to exec 'cc1plus'`
      *  Si se produce el error `gcc: error trying to exec 'cc1plus': execvp: No such file or directory`, instale los elementos esenciales de compilación mediante el comando `sudo apt-get install build-essential`.
      * Pase un nuevo nombre como primer parámetro a automl_setup para crear un nuevo entorno de Conda. Vea los entornos de Conda existentes mediante `conda env list` y quítelos con `conda env remove -n <environmentname>`.
      
* **Error de automl_setup_linux.sh**: si automl_setup_linus.sh produce el error `unable to execute 'gcc': No such file or directory`- en Ubuntu Linux:
  1. Asegúrese de que los puertos de salida 53 y 80 estén habilitados. En una máquina virtual de Azure, puede hacerlo desde Azure Portal. Para ello, seleccione la máquina virtual y haga clic en Redes.
  2. Ejecute el comando: `sudo apt-get update`
  3. Ejecute el comando: `sudo apt-get install build-essential --fix-missing`
  4. Vuelva a ejecutar `automl_setup_linux.sh`.

* **Error de configuration.ipynb**:
  * En el caso de Conda local, asegúrese primero de que automl_setup se haya ejecutado correctamente.
  * Asegúrese de que subscription_id es correcto. Busque subscription_id en Azure Portal. Para ello, seleccione Todos los servicios y, después, Suscripciones. Los caracteres "<" y ">" no deben incluirse en el valor de subscription_id. Por ejemplo, `subscription_id = "12345678-90ab-1234-5678-1234567890abcd"` tiene el formato válido.
  * Garantice el acceso de tipo Colaborador o Propietario a la suscripción.
  * Compruebe que la región sea una de las regiones admitidas: `eastus2`, `eastus`, `westcentralus`, `southeastasia`, `westeurope`, `australiaeast`, `westus2` o `southcentralus`.
  * Asegúrese de acceder a la región mediante Azure Portal.
  
* **Error de `import AutoMLConfig`** : hubo cambios en el paquete de la versión 1.0.76 del aprendizaje automático automatizado que requieren la desinstalación de la versión anterior antes de actualizar a la nueva versión. Si se detecta `ImportError: cannot import name AutoMLConfig` después de actualizar a la versión v 1.0.76, o posterior, del SDK desde una versión anterior, resuelva el error ejecutando `pip uninstall azureml-train automl` y luego `pip install azureml-train-auotml`. El script automl_setup.cmd lo hace automáticamente. 

* **Error de workspace.from_config**: si se produce un error en la llamada ws = Workspace.from_config()':
  1. Asegúrese de que el cuaderno configuration.ipynb se ha ejecutado correctamente.
  2. Si el cuaderno se ejecuta desde una carpeta que no está en la carpeta donde se ejecutó `configuration.ipynb`, copie la carpeta aml_config y el archivo config.json que contiene en la nueva carpeta. Workspace.from_config lee el archivo config.json de la carpeta del cuaderno o su carpeta principal.
  3. Si se usa una suscripción, un grupo de recursos, un área de trabajo o una región nuevos, asegúrese de volver a ejecutar el cuaderno `configuration.ipynb`. Cambiar el archivo config.json directamente solo funcionará si el área de trabajo ya existe en el grupo de recursos especificado en la suscripción indicada.
  4. Si quiere cambiar la región, cambie el área de trabajo, el grupo de recursos o la suscripción. `Workspace.create` no creará ni actualizará un área de trabajo si ya existe, aunque la región especificada sea diferente.
  
* **Error del cuaderno de ejemplo**: si se produce un error en un cuaderno de ejemplo que indica que la propiedad, el método o la biblioteca no existen:
  * Asegúrese de que se ha seleccionado el kernel correcto en el cuaderno de Jupyter Notebook. El kernel se muestra en la parte superior derecha de la página del cuaderno. El valor predeterminado es azure_automl. El kernel se guarda como parte del cuaderno. Por lo tanto, si cambia a un nuevo entorno de Conda, tendrá que seleccionar el nuevo kernel en el cuaderno.
      * Para Azure Notebooks, debe ser Python 3.6. 
      * En el caso de entornos de Conda locales, debe ser el nombre del entorno de Conda que especificó en automl_setup.
  * Asegúrese de que el cuaderno es para la versión del SDK que está usando. Puede comprobar la versión del SDK si ejecuta `azureml.core.VERSION` en una celda de Jupyter Notebook. Puede descargar la versión anterior de los cuadernos de ejemplo de GitHub; para ello, haga clic en el botón `Branch`, seleccione la pestaña `Tags` y, después, seleccione la versión.

* **Error de `import numpy` en Windows**: algunos entornos Windows ven un error al cargar Numpy con la versión más reciente de Python: 3.6.8. Si ve este problema, pruebe con la versión 3.6.7 de Python.

* **Error de `import numpy`** : compruebe la versión de TensorFlow en el entorno de Conda de aprendizaje automático automatizado. Las versiones compatibles son < 1.13. Desinstale TensorFlow del entorno si la versión es >= 1.13. Puede comprobar la versión de TensorFlow y desinstalar como se indica a continuación:
  1. Inicie un shell de comandos y active el entorno de Conda donde están instalados los paquetes de aprendizaje automático automatizado.
  2. Escriba `pip freeze` y busque `tensorflow`. Si se encuentra, la versión indicada debe ser < 1,13.
  3. Si la versión indicada no es una versión compatible, escriba `pip uninstall tensorflow` en el shell de comandos y escriba "y" (sí) para confirmar la operación.
  
 * **Error de ejecución con `jwt.exceptions.DecodeError`** : mensaje de error exacto: `jwt.exceptions.DecodeError: It is required that you pass in a value for the "algorithms" argument when calling decode()`.

    En el caso de las versiones anteriores o iguales a la versión 1.17.0 del SDK, la instalación podría dar como resultado una versión no compatible de PyJWT. Compruebe la versión de PyJWT en el entorno de Conda de aprendizaje automático automatizado. Las versiones compatibles son < 2.0.0. Puede comprobar la versión de PyJWT como se indica a continuación:
    1. Inicie un shell de comandos y active el entorno de Conda donde están instalados los paquetes de aprendizaje automático automatizado.
    2. Escriba `pip freeze` y busque `PyJWT`; si se encuentra, la versión indicada debe ser < 2.0.0.

    Si la versión indicada no es una versión compatible:
    1. Considere la posibilidad de actualizar a la versión más reciente del SDK de AutoML: `pip install -U azureml-sdk[automl]`.
    2. Si eso no es viable, desinstale PyJWT del entorno e instale la versión correcta como se indica a continuación:
        - Ejecute `pip uninstall PyJWT` en el shell de comandos y escriba `y` para confirmar.
        - Realice la instalación mediante `pip install 'PyJWT<2.0.0'`.

## <a name="next-steps"></a>Pasos siguientes

+ Obtenga más información sobre [cómo y dónde implementar un modelo](how-to-deploy-and-where.md).

+ Obtenga más información sobre [cómo entrenar un modelo de regresión con aprendizaje automático automatizado](tutorial-auto-train-models.md) o [cómo entrenar con aprendizaje automático automatizado en un recurso remoto](how-to-auto-train-remote.md).

+ Obtenga información sobre cómo entrenar varios modelos con AutoML en [Acelerador de soluciones de muchos modelos](https://aka.ms/many-models).
