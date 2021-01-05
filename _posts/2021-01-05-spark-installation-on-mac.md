---
layout: post
title:  Spark installation on macOS
categories: [Big Data]
excerpt: Spark installation on macOS along with verification.
---
> `homebrew` is assumed to be installed.

# Install Java
```bash
brew cask install java
```
# Install Scala
```bash
brew install scala
```
# Install Spark
```bash
brew install apache-spark
```
# Install Python & libs
> It is assumed Anacaonda is installed along with Python3
```bash
conda install pyspark
```
# Setup env
```bash
echo "export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/"  >> ~/.zshrc
echo "export JRE_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/jre/"  >> ~/.zshrc

echo "export SPARK_HOME=/usr/local/Cellar/apache-spark/3.0.1/libexec"  >> ~/.zshrc
echo "export PATH=/usr/local/Cellar/apache-spark/3.0.1/bin:$PATH"  >> ~/.zshrc

echo "export PYSPARK_PYTHON=/usr/local/bin/python3"  >> ~/.zshrc
echo "export PYSPARK_DRIVER_PYTHON=jupyter"  >> ~/.zshrc
echo "export PYSPARK_DRIVER_PYTHON_OPTS='notebook'"  >> ~/.zshrc
source ~/.zshrc
```
# Verify installation
Launch `pyspark` from the directory containing iPython notebook
```bash
➜  pyspark
[I 18:33:57.084 NotebookApp] JupyterLab extension loaded from .../opt/anaconda3/lib/python3.7/site-packages/jupyterlab
[I 18:33:57.085 NotebookApp] JupyterLab application directory is .../opt/anaconda3/share/jupyter/lab
[I 18:33:57.090 NotebookApp] Serving notebooks from local directory: .../spark
[I 18:33:57.090 NotebookApp] The Jupyter Notebook is running at:
[I 18:33:57.090 NotebookApp] http://localhost:8888/?token=....
[I 18:33:57.090 NotebookApp]  or http://127.0.0.1:8888/?token=....
[I 18:33:57.090 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 18:33:57.127 NotebookApp] 
    
    To access the notebook, open this file in a browser:
        file:///..../Library/Jupyter/runtime/nbserver-6161-open.html
    Or copy and paste one of these URLs:
        http://localhost:8888/?token=....
     or http://127.0.0.1:8888/?token=....
```
Write a sample spark program & check the result
```python
import pyspark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName('HelloSpark').getOrCreate()
df = spark.sql(''' select 'SQL' as Hello ''')
df.show()
```
```bash
+-----+
|Hello|
+-----+
|  SQL|
+-----+
```