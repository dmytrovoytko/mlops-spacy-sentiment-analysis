# MLOps project Training and Deployment of Spacy model for Sentiment analysis

Pet project / Capstone project for DataTalks.Club MLOps ZoomCamp`24: 

Spacy model trained on dataset based on [Amazon Reviews'23](https://amazon-reviews-2023.github.io/) processed via my [Data Engineering project Amazon Reviews](https://github.com/dmytrovoytko/data-engineering-amazon-reviews) ETL.

![MLOps project Spacy model for Sentiment analysis](/screenshots/amazon-reviews-sentiment-mlops.png)

Project can be tested and deployed in cloud virtual machine (AWS, Azure, GCP), **GitHub CodeSpaces** (the easiest option, and free), or locally without GPU.

To reproduce and review this project it would be enough less than an hour, prepared dataset is not huge as original, so you don't need much disk space. For GitHub CodeSpace option you don't need to use anything extra at all - just your favorite web browser + GitHub account is totally enough.

## Problem statement

Modern technologies, social media, messengers and chat bots, including ChatGPT, trained us to "expect" almost immediate response. As a result slow response often becomes a way "out of business". E-commerce websites automated shopping processes, but response on customer/user feedback yet is not so fast as we'd like to have, however it's quite critical, agree?

An easy step to improve such communications would be a sentiment analysis of user's feedback to filter which messages need extra attention. And that's where Machine Learning could shine.

Can we make it happen with limited resources, so even blog could afford it without spending money on those 'chatgpt'-like platforms or resource demanding tech using TensorFlow transformers?

I decided to experiment with several fast and light ML NLP libraries that can run on CPU, so it could be deployed on inexpensive hosting supporting python apps:
- [NLTK Vader](https://www.nltk.org/howto/sentiment.html)
- [TextBlob](https://textblob.readthedocs.io/en/dev/)
- [SpaCy](https://spacy.io/)

Testing showed: yes, they are really fast and easy to implement, but accuracy is not very high - around 79-80%.

However, SpaCy is different - it can be trained, so let's do it!

What dataset can we use for this, with variety of measured feedback? Of course from Amazon - the Everything Store. 
In [my previous project](https://github.com/dmytrovoytko/data-engineering-amazon-reviews) I processed Amazon dataset 2023. Original dataset is huge, with millions of reviews on more than 30 categories - from Toys and Games to Clothing and Electronics, gigabytes of data.

For this project I chose to work with a much smaller subset - only years 2020-2022, Kindle Store books. Extracted and stored [here](https://github.com/dmytrovoytko/reviews-sentiment-dataset).

## 🎯 Goals

This is my MLOps project started during [MLOps ZoomCamp](https://github.com/DataTalksClub/mlops-zoomcamp)'24.

And **the main goal** is straight-forward: build an end-to-end Machine Learning project: 
- choose dataset
- load & analyze data, preprocess it
- train & test ML model
- create a model training pipeline
- deploy the model (as a web service)
- finally monitor performance
- And follow MLOps best practices!

Dataset **Reviews** (original) contains users' books reviews (title and text), each book rated from 1 to 5. I used book review texts rated 1-3 as examples of negative sentiment, and 4-5 as positive. IMHO "4" is a bit tricky, because many readers described partly negative reasons why they didn't give "5". Samples are [here](/train_model/data).

Thanks to MLOps ZoomCamp for the reason to learn many new tools! 

## :toolbox: Tech stack

- MLFlow for ML experiment tracking
- Prefect for ML workflow orchestration
- Docker and docker-compose

## 🚀 Instructions to reproduce

- [Setup environment](#hammer_and_wrench-setup-environment)
- [Dataset](#arrow_heading_down-dataset)
- [Train model](#train-model)
- [Test prediction service](#test-prediction-service)
- [Deployment and Monitoring](#deployment-and-monitoring)
- [Best practices](#best-practices)

### :hammer_and_wrench: Setup environment

1. Fork this repo on GitHub.
2. Create GitHub CodeSpace from the repo.
3. **Start CodeSpace**
4. Run `pipenv install --dev` to install required packages.
5. If you want to play with/develop the project, you can also install `pipenv run pre-commit install` to format code before committing to repo.

### :arrow_heading_down: Dataset

Dataset files are automatically downloaded from [this repo](https://github.com/dmytrovoytko/reviews-sentiment-dataset), they are in parquet format and ~20mb each.
If you want to work with additional files, you can put them into `./train_model/data/` directory and change function `load_data_from_parquet()` in `./train_model/orchestrate.py`.
Samples of each partition (by years) you can see in `./train_model/data/` directory.

Data preprocessing includes filtering out outliers - too short (<25) and too long (>3000) reviews. Majority of retings are positive (4 and 5), so I added balancing positive and negative sentiments to improve prediction accuracy.

![Dataset details](/screenshots/dataset-details.png)

### Train model

Run `bash run-train-model.sh` or go to `train_model` directory and run `python orchestrate.py`.
This will start Prefect workflow to
- load training data (2021)
- call `spacy_run_experiment()` with different hyper parameters
- load testing data (2022)
- call `spacy_test_model()` to measure model performance and calculate confusion matrix
- finally, call `run_register_model()` to register the best model, which will be saved to `./model` directory.

Spacy has its own pipeline management via `project.yml` and `config.cfg` files. It controls how many epochs to run and when to stop to prevent overfitting. I call its (cli api) commands and override some parameters to tune model for better performance. And MLflow tracks all experiments, saves parameters and artifacts. Then you're able to compare metrics, including visual form.

To explore results go to `train_model` directory and run `mlflow server`.

![MLFlow experiments training Spacy model for Sentiment analysis](/screenshots/mlflow-00.png)

![MLFlow experiments training Spacy model for Sentiment analysis](/screenshots/mlflow-03.png)

Prefect orchestration

![Prefect orchestration](/screenshots/prefect-orchestration.png)

### Test prediction service

Run `bash test-service.sh` or go to `prediction_service` directory and run `bash test-run.sh`.
This will copy best model and latest scripts, build docker image, run it, and make curl requests.
Finally docker container will be stopped.

![Testing prediction service in dockerl for Sentiment analysis](/screenshots/testing-prediction-service.png)

### Deployment and Monitoring

To deploy web service run `bash deploy-service.sh`.

Monitoring is under development yet (adding Evidently AI).

### Best practices

    * [x] Unit tests
    * [x] Integration test (== Test prediction service)
    * [x] Code formatter (isort, black)
    * [x] Makefile
    * [x] Pre-commit hooks 

## Current results of training

By tuning avalable spaCy hyper parameters I managed to achive 84% accuracy.

![Trained Spacy model for Sentiment analysis: results](/screenshots/spacy-train-model.png)

You can find additional information which parameners result better performance on [screenshots](/screenshots).

## Next steps

I plan to deploy it on my hosting and test performance with other Amazon reviews (other categories).

Stay tuned!

## Support

🙏 Thank you for your attention and time!

- If you experience any issue while following this instruction (or something left unclear), please add it to [Issues](/issues), I'll be glad to help/fix. And your feedback, questions & suggestions are welcome as well!
- Feel free to fork and submit pull requests.

If you find this project helpful, please ⭐️star⭐️ my repo 
https://github.com/dmytrovoytko/mlops-spacy-sentiment-analysis to help other people discover it 🙏

Made with ❤️ in Ukraine 🇺🇦 Dmytro Voytko
