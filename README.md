# e2e Data Project 

This is a simple end-to-end example of an MLOps project that uses the [Iris Dataset](https://scikit-learn.org/1.4/auto_examples/datasets/plot_iris_dataset.html).  
The goal of this project is to create a model that allows you to automatically classify flowers into different species based on their properties, and to have a [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipeline enabled that will allow you to easily track and deploy your code to different environments, such as Development and Production.  

As a part of this project, you will set up:
- A job for ingesting the Iris data into a feature table (which, simply speaking, is a Delta table with a Primary Key)
- A notebook for training a simple classification model and storing the model in the Unity Catalog
- A job that uses the model to run inferences on the top of newly collected (unidentified) flowers
- An MLflow 3.0 deployment job for automating the process of defining the model used for running the production inferences

In the "Notebooks" folder, you'll find three groups of Python notebooks:
1. **DataPreprocessing**: these are the notebooks that are used in the data preprocessing job. Within the "Jobs" folder, you will find a workflow called "batch-inference-job.yml" that basically triggers these notebooks using serverless compute. 
2. **ModelTrainingAndDeployment**: 
  - The "model-training.ipynb" notebook is supposed to be executed as a standalone notebook (meaning it should not be linked to any jobs). It features the code to train a simple classification model and save it to the Unity Catalog. It is important to mention that this model should be registered with the "Challenger" alias every time it gets trained. "Challenger" is a term that is used to define a model that has the potential of becoming the official model for running inferences, but not necessarily will become it, unless formally validated. 
  - The "model-evaluation.ipynb", "model-approval.ipynb" and "model-deployment.ipynb" notebooks are a part of the "model-deployment-job" workflow, which can be found in the "Jobs" folder. These notebooks and job were inspired by the [Deployment Jobs](https://docs.databricks.com/gcp/en/mlflow/deployment-job) concept, introduced in MLflow 3.0. The idea is to add a series of steps that include metrics evaluation and human-in-the-loop approval before effectively deploying a model for production usage (or making it the *Champion* model, for instance). As an important note: after you save the first version of your model to the UC, you need to [connect your model to the deployment job](https://docs.databricks.com/gcp/en/mlflow/deployment-job#connect-the-deployment-job-to-a-model).

<img src="https://docs.databricks.com/gcp/en/assets/images/deployment-job-create-ui-c43b64da503e3f0babcb9ff81a78610d.png" alt="Deployment Jobs in the Databricks UI" width="600"/>  

3. **Inference**: 
  - The "batch-inference.ipynb" notebook is used in the "batch-inference-job.yml" job, which can be found in the Jobs folder. It is a simple job that uses the *Champion* model for running batch inference on the top of new Iris samples and save the results to an Inference Table. 
  - The "realtime-inference.ipynb" notebook is just there to show an example of how you can use a Serving Endpoint that is connected to your *Champion* model to run inferences in near-real-time (note: this serving endpoint gets created programatically in the "model-deployment.ipynb" notebook)

Now, talking about the other elements from this repository in more depth:
  - The file **databricks.yml** is a file that defines how to deploy all of these jobs in an automatic fashion using [Databricks Asset Bundles](https://docs.databricks.com/aws/en/dev-tools/bundles/). The jobs will be parameterized based on whether they are *development* or *production* jobs. By running a simple ```databricks bundle deploy --target <dev, prod>``` command, all of those jobs should be automatically created within your workspace (provided you have the necessary permissions to create these resources), pointing to the right resources depending on each environment.
  - The file **.github/workflows/databricks-deployment.yml** specifies the *Continuous Deployment (CD)* piece of our work if you use GitHub as your CI/CD tool. You can set up commands to run whenever changes occur to your codebase by leveraging [GitHub Actions](https://github.com/features/actions). For instance, if a push occurs to the dev branch of this repository, the command `databricks bundle deploy --target dev` will run automatically; and if a push is made to the main branch, the `databricks bundle deploy --target prod` command will get executed. 
  - The file **azure-pipelines.yml** specifies the *Continuous Deployment (CD)* piece of our work if you use Azure DevOps as your CI/CD tool. You can set up commands to run whenever changes occur to your codebase by leveraging [Pipelines](https://azure.microsoft.com/en-us/products/devops/pipelines). For instance, if a push occurs to the *dev* branch of this repository, the command `databricks bundle deploy --target dev` will run automatically; and if a pull request is made to the *master* branch, the `databricks bundle deploy --target prod` command will get executed. 

Having a *Continuous Deployment (CD)* pipeline enabled means that if you make any changes to your dev branch, your dev pipelines will be updated; and if you make any changes to your master branch, your prod pipelines will be automatically updated. 

Let us know if you need support in your MLOps journey! 

Contacts: pedro.zanlorensi@databricks.com